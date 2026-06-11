### Tools
tmux
wget
alpine requires repo # removal change
alpine uses apk
ubuntu uses apt

### rsync command
keeps content synced
```bash
rsync -avz --progress -e "ssh -p 8022" \
  --exclude='node_modules' \
  --exclude='.git' \
  --exclude='dist' \
  /path/to/source/ user@alpine-ip:/path/to/destination/
```

### fswatch


### conda
![[Pasted image 20260514023342.png]]


