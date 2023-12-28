Rsync (Remote Sync) is a powerful and versatile file synchronization tool commonly used in Unix and Linux systems. Here's a cheat sheet to help you use rsync effectively:

### Basic Syntax:

```bash
rsync [options] source destination
```

### Common Options:

- `-a, --archive`: Archive mode; equals `-rlptgoD` (recursive, links, permissions, times, group, owner, devices).
- `-v, --verbose`: Increase verbosity; shows files as they are transferred.
- `-r, --recursive`: Recursively copy directories.
- `-u, --update`: Skip files that are newer on the receiver.
- `-l, --links`: Copy symlinks as symlinks.
- `-p, --perms`: Preserve permissions.
- `-t, --times`: Preserve modification times.
- `-g, --group`: Preserve group.
- `-o, --owner`: Preserve owner.
- `-D`: Preserve special and device files.
- `--delete`: Delete extraneous files from the destination.
- `--exclude=PATTERN`: Exclude files/folders matching the pattern.

### Transfer Options:

- `-z, --compress`: Compress file data during the transfer (prefer to use over network transfer).
- `--progress`: Show progress during transfer.
- `--partial`: Keep partially transferred files.
- `--bwlimit=RATE`: Limit I/O bandwidth; e.g., `--bwlimit=1000` limits to 1000 KB/s.
- `--timeout=SECONDS`: Set I/O timeout.

### Connection Options:

- `-e, --rsh=COMMAND`: Specify the remote shell command (default is "ssh").
- `-p, --port=PORT`: Specify the remote shell port.

### Examples:

1. **Local to Local:**
   ```bash
   rsync -av /path/to/source /path/to/destination
   ```

2. **Local to Remote:**
   ```bash
   rsync -av /path/to/source username@remote:/path/to/destination
   ```

3. **Remote to Local:**
   ```bash
   rsync -av username@remote:/path/to/source /path/to/destination
   ```

4. **Exclude files or directories:**
   ```bash
   rsync -av --exclude=*.log /path/to/source /path/to/destination
   ```

5. **Delete extraneous files at the destination:**
   ```bash
   rsync -av --delete /path/to/source /path/to/destination
   ```

6. **Using SSH Key for Authentication:**
   ```bash
   rsync -av -e "ssh -i /path/to/private-key.pem" /path/to/source username@remote:/path/to/destination
   ```

7. **Limit Bandwidth:**
   ```bash
   rsync -av --bwlimit=1000 /path/to/source /path/to/destination

1000 implies in 1000K bytes.
   ```


8. **Show Progress:**
   ```bash
   rsync -av --progress /path/to/source /path/to/destination
   ```

Remember to replace placeholders such as `/path/to/source`, `/path/to/destination`, `username`, and `remote` with your actual paths, usernames, and remote addresses. Adjust options based on your specific requirements.