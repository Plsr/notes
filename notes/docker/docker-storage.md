# Docker Storage
Link to Docs: https://docs.docker.com/engine/storage/

- By default, all files created inside a container are stored on the writable container layer (which sits on top of the read-only image layer)
- Data written to the container layer is destroyed with the container
- To offer getting data out of the writable container layer, docker offers storage mounts

## Storage mount options
### Volume mounts
- Volume data is stored on the filesystem of the host
- It persists after the container is destroyed
- Volumes can only be accessed using a container
- Managed by the daemon host, offers great performance

### Bind mounts
- Create a direct link between a host system path and a container
- Not isolated by Docker, so both the host and container can access and modify data

### tmpfs mounts
- live in memory, no data is written to disk
- data is lost when either container or host are stopped/restarted

### Named Pipes
- No idea about those
