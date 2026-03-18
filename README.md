# 5a_Create_Socket_for_HTTP_for_webpage_upload_and_download
## AIM :
To write a PYTHON program for socket for HTTP for web page upload and download
## Algorithm

1.Start the program.
<BR>
2.Get the frame size from the user
<BR>
3.To create the frame based on the user request.
<BR>
4.To send frames to server from the client side.
<BR>
5.If your frames reach the server it will send ACK signal to client otherwise it will send NACK signal to client.
<BR>
6.Stop the program
<BR>

## Program 

server.py

```
import socket
import os

host = "localhost"
port = 12345

srv = socket.socket()
srv.bind((host, port))
srv.listen(1)

print("Server started and waiting for client...")

connection, address = srv.accept()
print("Client connected:", address)

while True:
    cmd = connection.recv(1024).decode()

    if not cmd:
        break

    if cmd == "UPLOAD":

        fname = connection.recv(1024).decode()
        size = int(connection.recv(1024).decode())

        with open("srv_" + fname, "wb") as f:
            while True:
                packet = connection.recv(size)

                if packet == b"END":
                    break

                if packet:
                    f.write(packet)
                    connection.send(b"ACK")
                else:
                    connection.send(b"NACK")

        print("Upload completed at server")

    elif cmd == "DOWNLOAD":

        fname = connection.recv(1024).decode()
        path = "srv_" + fname

        if os.path.isfile(path):
            connection.send(b"FOUND")

            size = int(connection.recv(1024).decode())

            with open(path, "rb") as f:
                block = f.read(size)

                while block:
                    connection.send(block)

                    response = connection.recv(1024).decode()

                    if response == "ACK":
                        block = f.read(size)
                    else:
                        print("Frame resend requested")

            connection.send(b"END")
            print("File sent to client")

        else:
            connection.send(b"NOTFOUND")

    elif cmd == "EXIT":
        print("Client disconnected")
        break

connection.close()
srv.close()
```

client.py

```
import socket

host = "localhost"
port = 12345

cli = socket.socket()
cli.connect((host, port))

while True:

    print("\nMENU")
    print("1 - Upload HTML file")
    print("2 - Download HTML file")
    print("3 - Quit")

    option = input("Enter option: ")

    if option == "1":

        fname = input("Enter file name to upload: ")
        size = int(input("Enter frame size: "))

        cli.send(b"UPLOAD")
        cli.send(fname.encode())
        cli.send(str(size).encode())

        with open(fname, "rb") as f:

            chunk = f.read(size)

            while chunk:
                cli.send(chunk)

                reply = cli.recv(1024).decode()

                if reply == "ACK":
                    print("Frame delivered")
                else:
                    print("Error in frame")

                chunk = f.read(size)

        cli.send(b"END")
        print("File upload finished")

    elif option == "2":

        fname = input("Enter file name to download: ")
        size = int(input("Enter frame size: "))

        cli.send(b"DOWNLOAD")
        cli.send(fname.encode())

        status = cli.recv(1024).decode()

        if status == "FOUND":

            cli.send(str(size).encode())

            with open("client_" + fname, "wb") as f:
                while True:

                    packet = cli.recv(size)

                    if packet == b"END":
                        break

                    f.write(packet)
                    cli.send(b"ACK")

            print("File downloaded successfully")

        else:
            print("Requested file not available on server")

    elif option == "3":

        cli.send(b"EXIT")
        break

    else:
        print("Wrong option")

cli.close()

```

## OUTPUT

sever.py

<img width="1092" height="216" alt="Screenshot 2026-03-18 083254" src="https://github.com/user-attachments/assets/5902ec48-4ffb-45b1-890f-3c628faccab0" />

client.py

<img width="1265" height="425" alt="Screenshot 2026-03-18 083242" src="https://github.com/user-attachments/assets/808e5421-22e9-460d-9ac0-6ae139972c09" />

## Result
Thus the socket for HTTP for web page upload and download created and Executed
