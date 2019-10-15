# Snips Platform Docker Container

## Scope

This repository provides the instructions, DockerFile and scripts needed to:

- Run the snips platform components in a docker container.
- Provide the host platform audio capture and playback interfaces to the container.

## Build the snips platform container

### Prequisites

- [Docker](https://docs.docker.com/ee/desktop/) is up and running
- This repository is available on your host environment.

### Build the container

Enter the `docker-pulseaudio` sub folder and build the container.

```bash
cd docker-pulseaudio
docker build --no-cache -t snips-pulseaudio-docker .
```

Once completed, this operation should return the following message.

```bash
Successfully tagged snips-pulseaudio-docker:latest
```

This container is preconfigured for the `snips-audio-server` component to use `PulseAudio` audio backend.

Hence, a `PulseAudio` **server** must be started on the host operating system to share its capture and playback interfaces with the container running the snips platform.

```ascii
+ ----------------------------------------------------------- +
| Host OS                         +------------------------+  |
|                                 | Docker Container       |  |
| +----------------------+  tcp   | +--------------------+ |  |
| | PulseAudio Server    | <----> | | PulseAudio Client  | |  |
| +----------------------+        | +--------------------+ |  |
|          |  ^ ️                  |         |  ^           |  |
|      ️    v  |                   |         v  |           |  |
| +----------------------+        | +--------------------+ |  |
| | Host Audio           |        | | snips-audio-server | |  |
| | Capture and Playback |        | +--------------------+ |  |
| +----------------------+        +------------------------+  |
+ ----------------------------------------------------------- +
```

## Configure PulseAudio server

### Install on Windows

- Retrieve Windows [v1.1 binaries](https://www.freedesktop.org/wiki/Software/PulseAudio/Ports/Windows/Support/)
- Unzip the content of the archive.
- Open a terminal and navigate to the `bin` subfolder.

### Install on MacOS (homebrew)

```bash
brew install pulseaudio
```

### Run PulseAudio Server

The host capture and playback interface will be interfaced to the container using `module-native-protocol-tcp` PulseAudio module.

Execute the following command in a terminal to launch the server as backgroung process

```bash
# Windows
pulseaudio.exe --load="module-native-protocol-tcp auth-anonymous=1" --exit-idle-time=-1 --daemon
```

```bash
# MacOS and Linux
pulseaudio --load="module-native-protocol-tcp auth-anonymous=1" --exit-idle-time=-1 --daemon
```

Notes:

- Using Windows or MacOS make sure that the PulseAudio server is allowed to use the network and audio interfaces.
- Using Windows, the PulseAudio Server can be terminated using the task manager.
- For debug purpose, remove `--daemon` flag to observe the server logs in a terminal.
- In this example, no authentication is needed to connect to the PulseAudio Server. Please refer to PulseAudio [documentation](https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Network/#index2h2) for other authentication methods.

## Run the Snips Platform docker container

### PulseAudio Server IP Address

The host IP address must be passed to the container for it to connect to the PulseAudio server.

Identify the host IP address and replace the `<HOST_IP_ADDRESS>` place holder by the correct value.

### Start the docker container

```bash
docker run -it -e PULSE_SERVER=<HOST_IP_ADDRESS> -e SNIPS_AUDIO_SERVER_ARGS="--alsa_capture=pulse --alsa_playback=pulse -v" -e SNIPS_AUDIO_SERVER_ENABLED="true" snips-pulseaudio-docker:latest
```

### Inspect the platform with snips-watch

You can then connect to the running container and start `snips-watch` to check if everything is working as expected.

To do so, find your snips container `Id` or `name` and start `snips-watch`.

```bash
docker ps -a
docker exec -it <ID> snips-watch -v
```

## Deploy your assistant

You might want to replace the default weather demo assistant with your own assistant.

To do so, retrieve an assistant package on the console and unzip it on your machine.

### Mount your assistant in your snips platform container

To provide this assistant to your container without modifiying it, execute the following command to override the default assistant subfolder located at `/usr/share/snips/assistant`.

Replace `<PATH/TO/YOUR/ASSISTANT>` placeholder by the correct path to your assistant.

```bash
docker run -it -e PULSE_SERVER=<HOST_IP_ADDRESS> -e SNIPS_AUDIO_SERVER_ARGS="--alsa_capture=pulse --alsa_playback=pulse -v" -e SNIPS_AUDIO_SERVER_ENABLED="true" -v <PATH/TO/YOUR/ASSISTANT>:/usr/share/snips/assistant  snips-pulseaudio-docker:latest
```

### Build a new container with your assistant

If you want a single container that include your assistant, modify the docker file to `COPY` it to your container at build time.

To do so, modify the `Dockerfile` to copy your assistant folder into the container.

If your assistant folder is located in the `docker-pulseaudio` sub folder, add this line to your `Dockerfile`.

```bash
COPY assistant /usr/share/snips/assistant
```

Once done, rebuild your docker container and start it.
