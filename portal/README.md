# Portal/Service registry

Optional portal routing to internal services to avoid having to memorise every single exposed port. Note: is hardcoded to the wee-man hostname

Is currently containerised for no real reason, but it can probably be included in the compose file later

Build the image for your arch

`docker buildx build --platform linux/amd64 -t wmportal --load .`

Start the container manually on your favourite port (or add to the compose file)

`docker run -d -p 8081:8081 --name wmportal wmportal:latest`
