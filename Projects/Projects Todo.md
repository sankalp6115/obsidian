### Sound-Based Triggers (like the clap)
-   **Clap once → open workspace**
-   **Clap twice → lock screen**
-   **Finger snap → toggle music**
-   **Whistle → trigger custom script**

Implementation direction:
-   Use Python + `pyaudio` / `sounddevice`
-   Detect amplitude spikes or pattern matching
-   Run shell scripts (`open -a`, `osascript`, etc.
### Personal Search Engine for Your Life
Search across:
-   files
-   notes
-   browsing history
-   code
Build:
-   index everything locally
-   fast search engine
Concepts
-   inverted index
-   ranking algorithms

