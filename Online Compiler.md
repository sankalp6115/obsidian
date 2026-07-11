User types code  
│  
▼  
Browser sends source code  
│  
▼  
Backend receives request  
│  
▼  
Create isolated Docker container / microVM  
│  
▼  
Write source code to a file  
│  
▼  
Compile (if needed)  
│  
▼  
Execute program  
│  
├── Enforce CPU limit  
├── Enforce memory limit  
├── Enforce execution timeout  
├── Restrict filesystem  
└── Optionally disable network  
│  
▼  
Capture stdout and stderr  
│  
▼  
Destroy the sandbox  
│  
▼  
Return JSON response  
│  
▼  
Browser displays the output