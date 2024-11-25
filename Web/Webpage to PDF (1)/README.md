# Webpage to PDF (1) ✅

## Challenge Description
> ![image](https://github.com/user-attachments/assets/5f8255b0-539c-4089-ab80-d303c2779e28)


#### Challenge Overview  
The challenge is a web application that converts a webpage (given its URL) into a PDF. The application:  
1. Takes a URL input.  
2. Downloads the page as an HTML file.  
3. Uses `wkhtmltopdf` to convert the HTML to a PDF file, saved using a `session_id` from cookies.

   ![image](https://github.com/user-attachments/assets/f7167c10-1425-4921-bc15-45372bca1375)


**Goal**: Read the flag stored in `/flag.txt`.  

---

### Steps to Solve  

#### 1. Observing the Website  
- Input a URL (e.g., `https://example.com`) and get a PDF.  
- The `session_id` in cookies determines the file name of the PDF and HTML.  

#### 2. Reviewing the Code  
- The server executes the command:  
  ```bash
  wkhtmltopdf {session_id}.html {session_id}.pdf
  ```  
- The `execute_command.py` uses `shlex.split()` to sanitize arguments.  

#### 3. Target: Read `/flag.txt`  
The flag file is stored in `/flag.txt`. Since `wkhtmltopdf` processes local files, we aim to read it using Local File Inclusion (LFI).  

---

### Exploitation  

#### Step 1: Set Up a Malicious Website  
- Create a webpage with this HTML:  
  ```html
  <iframe src="file:///flag.txt" height="500" width="500"></iframe>
  ```  
- Host it using a simple service like JS Bin or any HTML playground.

#### Step 2: Bypass Restrictions  
- `wkhtmltopdf` blocks access to local files by default. To bypass, use the `--enable-local-file-access` parameter.  

#### Step 3: Inject Parameters  
1. Modify the `session_id` cookie to include the argument:  
   ```
   --enable-local-file-access
   ```  
2. The resulting command becomes:  
   ```bash
   wkhtmltopdf --enable-local-file-access session_id.html session_id.pdf
   ```  

#### Step 4: Get the Flag  
- Submit the malicious webpage URL.

  ![image](https://github.com/user-attachments/assets/16b7975f-3a11-4a7e-b92e-707ddf737ec0)

- Remove the `--enable-local-file-access%20`
- Download the generated PDF to read `/flag.txt`.

  ![image](https://github.com/user-attachments/assets/c67e8396-3d8f-426d-bf2d-f1ec5a33a4c3)


---

### Flag  
By combining LFI with command-line injection, the flag is retrieved:  
```
hkcert24{h0w-t0-use-AI-wisely-and-s4fe1y?}
```  
