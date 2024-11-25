# Custom Web Server (1) ❌

## Challenge Description
> ![image](https://github.com/user-attachments/assets/6c6d3faa-1507-44b1-ab11-abcead353e6a)


## Approach
1. You are given a website, the target is to play a game to have score over 300.
2. Notice there are players with super high score, thus, all we need to do is to manipulate the score and obtain the flag.

   ![image](https://github.com/user-attachments/assets/68355a4c-4745-463a-98b6-07499b92669c)

3. Register and login to play the game.

   ![image](https://github.com/user-attachments/assets/4fca4895-074f-46da-abb0-f88cb7e9b382)

4. After you finish the game, The score will be updated to database via /update_score.php. Replay the game and turn on intercept. Intercept /update_score.php and ready to edit the POST request.

   ![image](https://github.com/user-attachments/assets/03c34f2e-2f98-4ce5-977a-575b0eaf04ab)

5. Review sourcecode of /game.php (either Ctrl + U, or F12), find out how the hash is calculated.

   ![image](https://github.com/user-attachments/assets/559536e1-669f-4555-8930-d5f6b8c7f7ce)

6. The request body is SHA-256 hash of concatinating secretKey, username, and score.

    ![image](https://github.com/user-attachments/assets/16777ba1-8453-45dc-b1a0-9c6e31666bc8)

7. Use online [SHA-256 calculator](https://emn178.github.io/online-tools/sha256.html) to change the hash: secretKey + username + score: *e.g. 3636f69fcc3760cb130c1558ffef5e24tuah3000*

   ![image](https://github.com/user-attachments/assets/7efbdac4-75d0-43da-94f8-b05f12ea6843)

8. Go back to Burpsuite and change the hash correspondingly. Remember to change the score and hash in request body. Then, forward the request.

   ![image](https://github.com/user-attachments/assets/be0db6da-8da5-4c4f-ac2c-f5b2f7fe0ef8)

10. Head to the scoreboard page to obtain the flag.

    ![image](https://github.com/user-attachments/assets/d37b4e35-d713-4c69-aa10-14af4f56f756)

## Flag: 
hkcert24{r3d33m_f0r_4_fr33_lunch}
