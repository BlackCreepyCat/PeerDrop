<img width="1357" height="819" alt="image" src="https://github.com/user-attachments/assets/be637c34-e782-4c53-863c-61eb1c43a8f0" />

<img width="1358" height="821" alt="image" src="https://github.com/user-attachments/assets/1bdbbbab-31a7-4756-9aa9-56d4bd47087e" />


# PeerDrop
 
A single-file, self-contained web app for direct peer-to-peer file transfer between browsers : no backend, no upload, no file size limit.
 
PeerDrop uses [PeerJS](https://peerjs.com/) (WebRTC) to open a direct data channel between two browsers. Files never touch a server: they stream straight from sender to receiver, chunk by chunk, over an encrypted WebRTC connection.
 
## Features
 
- **Direct browser-to-browser transfer** : files are streamed over WebRTC data channels, not uploaded anywhere.
- **No file size limit** : large files are sent as a stream of 256 KB chunks instead of being loaded into memory as a whole.
- **Server mode** : one peer hosts a file library that one or more receivers can browse and download from, in parallel, without a re-upload for each connection.
- **Receive mode** : connect to a server using a 6-character code and pick which of the hosted files to download.
- **Save directly to disk** : in browsers that support the File System Access API (Chromium-based, e.g. Edge or Chrome), the receiver can pick a destination folder and files are written straight to disk without a save dialog per file. Other browsers fall back to a standard download per file.
- **Resumable transfers** : if the connection drops mid-file, PeerDrop automatically reconnects and resumes from the last received byte instead of restarting.
- **Skip already-downloaded files** : when a destination folder is selected, files already present on disk (matching name and size) are detected and can be skipped automatically.
- **Integrity checking** : optional SHA-256 hashing lets the receiver verify that a file arrived byte-for-byte identical to the original (disabled automatically above 500 MB to avoid excessive memory use).
- **Live progress & ACK tracking** : the server sees, per connected receiver, which files have been fully acknowledged as received, plus live transfer speed, ETA, and connection ping (RTT).
- **Automatic reconnection** : both server and receiver retry the connection (up to 8 attempts) if it's interrupted, and pause/resume the affected transfer instead of failing it.
- **Network-aware** : the app detects when the browser goes offline and pauses transfers instead of losing them, resuming automatically once connectivity returns.
- **Safe downloads** : potentially executable file types (`.html`, `.js`, `.svg`, `.exe`, `.sh`, `.php`, etc.) are always served as generic binary downloads (`application/octet-stream`), never with a MIME type the browser could execute.
- **Multi-language UI** : English, French, Italian, Portuguese, Spanish, and Russian, detected from the browser and switchable at any time (saved locally).
- **No installation, no dependencies to manage** : it's a single HTML file. Open it in a browser and it works.
## How it works
 
PeerDrop only uses WebRTC to establish a direct connection between two browsers. A public PeerJS signaling server is used solely to help the two browsers discover each other and negotiate the connection (an "ICE handshake") : once that handshake succeeds, all file data flows directly between the two peers, not through the signaling server. No file content is ever uploaded to a server or stored anywhere outside the two browsers involved in the transfer.
 
There are two roles:
 
### Server mode
The peer who wants to share files.
 
1. Open PeerDrop and choose **Server**.
2. A 6-character connection code is generated automatically.
3. Drag and drop files onto the server screen (or click to browse) to add them to the shared library. Files can be added or removed at any time : connected receivers are notified instantly.
4. Share the 6-character code with whoever should receive the files.
5. As receivers connect, they appear in the **Receivers** panel with live download progress and per-file ACK (acknowledgment) status, confirming exactly which files each receiver has fully downloaded.
6. Multiple receivers can connect and download at the same time, each choosing their own subset of files.
### Receive mode
The peer who wants to download files.
 
1. Open PeerDrop and choose **Receive**.
2. Enter the 6-character code given by the server and click **Connect**.
3. (Optional, recommended) Choose a destination folder. If the browser supports it, files save directly to that folder with no per-file save dialog. If no folder is chosen, each file triggers a standard browser download instead.
4. Browse the list of files the server is hosting, check the ones you want, and click **Download selected**. Files already fully present in the chosen folder are marked and can be skipped automatically.
5. If the connection drops during a transfer, PeerDrop reconnects automatically and resumes the file exactly where it left off.
## Getting started
 
PeerDrop is a single HTML file with no build step and no server-side component required for the file transfer itself.
 
1. Download `p2ptransfer.html` (or clone this repository).
2. Open the file directly in a browser, or serve it from any static web host / local web server.
3. Share the URL (or the file) with the person on the other end : one of you picks **Server**, the other picks **Receive**.
> **Note:** the file loads the PeerJS library from a CDN (by default `https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js` : see [PeerJS](https://peerjs.com/) for other CDN options, e.g. jsDelivr). An internet connection is required for the initial connection handshake between peers, even though the actual file transfer afterward is direct peer-to-peer.
 
### Recommended browser
 
Chromium-based browsers (Microsoft Edge, Google Chrome, etc.) are recommended, since they support the File System Access API : this lets received files be written directly to a chosen folder without a save dialog for every single file. Other browsers still work fully, but fall back to a standard download prompt per file.
 
## Technical notes
 
- **Transport:** WebRTC `DataChannel`, via [PeerJS](https://peerjs.com/) with binary serialization.
- **Chunk size:** 256 KB per chunk during transfer.
- **Hashing:** SHA-256, computed via the browser's `SubtleCrypto` API, in 4 MB slices; automatically skipped for files/blobs larger than 500 MB.
- **Max simultaneous receivers per server:** 20 (basic DoS protection).
- **Heartbeat:** server and receiver exchange ping/pong messages (every 4 seconds, 9-second timeout) to detect a dead connection faster than WebRTC's own state reporting.
- **Persisted locally:** only the UI language preference (`localStorage`), nothing related to file content.
- **No backend code** in this repository : the signaling server is the public PeerJS cloud service; everything else runs entirely client-side.
## Limitations
 
- Both peers must have the page open in a browser at the same time for a transfer to happen (it is not an async "drop and leave" service).
- The initial connection depends on an internet connection to the PeerJS signaling server; if that server is unreachable, peers cannot discover each other (though once connected, NAT traversal may still occasionally require a TURN relay depending on network configuration).
- SHA-256 verification is disabled above 500 MB per file to avoid excessive browser memory usage.
