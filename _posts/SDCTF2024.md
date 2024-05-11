#SDCTF 2024 Writeup
###About the Author
#####[Bradley Greer](https://www.linkedin.com/in/greerbk/ "LinkedIn")
- Recent Graduate of Bloomsburg University of Pennsylvania
- Major in Digital Forensics and Cybersecurity
- SOC Analyst at MSP


**Table of contents**
[TOC]
##SDES
*I heard DES was broken, so I wrote SDES - San Diego Encryption Standard. It has a key space of 2^64 - no one's ever going to break it... right?*
[SDES.py](https://www.linkedin.com/in/greerbk/ "SDES.py")
[server.py](https://www.linkedin.com/in/greerbk/ "server.py")
---------
the cipher uses a key-dependent dynamic configuration of S-boxes, rotating with each encryption based on the key.
A good way to start a challenge like this is to see how it encrypts all 0's, and to see if its different every time. *it is.*
With this information we can try 00000000000000 then 01010101010101.
Okay what if we do 0001?
![](https://pandao.github.io/editor.md/images/logos/editormd-logo-180x180.png)
Realizing it only encrypts byte by byte, I wrote a code that will netcat into this server encrypt all 00 - FF repeats, put it into a text file, and then give me the message to decrypt.
With all 256 possible values for each character space encrypted, we can then compare the value from the message to decrypt and byte for byte decrypt it
```python
import socket
import time

def send_command(s, cmd, file, expect_response=True):
    print("Sending:", cmd)
    s.sendall(cmd.encode())
    if expect_response:
        time.sleep(1)  # Wait for server to process the command
        data = s.recv(1024)  # Adjust buffer size if necessary
        response = data.decode()
        print("Received:", response)
        return response
    else:
        # Here we'll also wait for a response but just not store it
        time.sleep(1)
        data = s.recv(1024)
        response = data.decode()
        print(response)  # Printing response even when not expected to store
        return response

def main():
    host = '192.168.56.1'
    port = 19732

    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((host, port))
        with open("encryption_responses.txt", "w") as file:
            # Initial connection and read welcome messages
            data = s.recv(1024)
            initial_message = data.decode()
            print(initial_message)

            # Send 255 encryption commands
            for i in range(256):
                hex_value = f"{i:02X}" * 8
                send_command(s, 'E\n', file)  # Choose to encrypt
                response = send_command(s, hex_value + '\n', file)  # Send the hex string to encrypt
                if "Encrypted message:" in response:
                    encrypted_msg = response.split('\n')[1].strip()
                    file.write(f"{encrypted_msg} - {hex_value}\n")

            # All encryption done, now moving to decryption
            print("Sending decryption command...")
            decryption_prompt = send_command(s, 'D\n', file, expect_response=True)
            print("Decryption challenge response:", decryption_prompt)
            # Wait for user input to attempt decryption
            user_input = input("Enter your guess for the original plaintext (in hex form): ")
            send_command(s, user_input + '\n', file, expect_response=False)  # Now it will print response

if __name__ == "__main__":
    main()


if __name__ == "__main__":
    main()
```
