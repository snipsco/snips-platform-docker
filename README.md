# Snips Platform Docker Container

## Scope

This repository provides the instructions, DockerFile and scripts needed to:

- Run the snips platform components in a docker container.
- Provide the host platform audio capture and playback interfaces to the container.

> Those instructions assume that the reader is familiar with the use of docker containers.

## Build the snips platform container

### Prequisites

- [Docker](https://docs.docker.com/ee/desktop/) is up and running on your host operating system.

### Clone this repository

```bash
git clone git@github.com:snipsco/snips-platform-docker.git
```

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

- Using Windows or MacOS, make sure that the PulseAudio server is allowed to use the network and audio interfaces.
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

> **Note**
>
> On Windows, when starting the container, the following error log appears  when the entry point script as been modified with windows end of line caracter: `standard_init_linux.go:211: exec user process caused "no such file or directory"`
>
> Use a tool such as `dos2unix` to reformat `snips-entrypoint.sh` correctly and rebuild your container.

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

Then, you can either mount your assistant folder within the snips-platform container or build a new container to include your assistant.

### Method 1 - Mount your assistant in your snips platform container

To provide this assistant to your container without modifiying it, execute the following command to override the default assistant subfolder located at `/usr/share/snips/assistant`.

Replace `<PATH/TO/YOUR/ASSISTANT>` placeholder by the correct path to your assistant.

```bash
docker run -it -e PULSE_SERVER=<HOST_IP_ADDRESS> -e SNIPS_AUDIO_SERVER_ARGS="--alsa_capture=pulse --alsa_playback=pulse -v" -e SNIPS_AUDIO_SERVER_ENABLED="true" -v <PATH/TO/YOUR/ASSISTANT>:/usr/share/snips/assistant  snips-pulseaudio-docker:latest
```

### Method 2 - Build a new container with your assistant

If you want a single container that include your assistant, modify the docker file to `COPY` it to your container at build time.

To do so, modify the `Dockerfile` to copy your assistant folder into the container.

If your assistant folder is located in the `docker-pulseaudio` sub folder, add this line to your `Dockerfile`.

```bash
COPY assistant /usr/share/snips/assistant
```

Once done, rebuild your docker container and start it.

## Integration with your action code

There is also multiple ways to proceed, one can either:

- Expose mosquitto MQTT broker port to the host operating system and run the action code locally.
- Package and run the action code within the container.

### Method 1 - Expose the MQTT broker to the host

Mosquitto MQTT broker is already running on the container on the port `1883`.

To expose this port to the host, modify the DockerFile, `EXPOSE` this port and rebuild the container.

```bash
# e.g. expose port 1883 using port 8888 on the host at build time.
EXPOSE 8888:1883
```

It is also possible to do it at execution time by adding the option `-p <target_port>:<source_port>` to the command line.

```bash
# e.g. expose port 1883 using port 8888 on the host at execution time.
docker run -it -e PULSE_SERVER=<HOST_IP_ADDRESS> -e SNIPS_AUDIO_SERVER_ARGS="--alsa_capture=pulse --alsa_playback=pulse -v" -e SNIPS_AUDIO_SERVER_ENABLED="true" -p 8888:1883 snips-pulseaudio-docker:latest
```

Then you can test that the MQTT broker is correctly exposed on your local host using a MQTT client such as `mosquitto_pub`

```bash
mosquitto_pub -t hermes/tts/say -h localhost -p 8888 -m '{"siteId":"default", "lang":"en-us", "text": "Can you ear this? Thanks for your attention.", "id": "123456", "sessionId": "1234"}'
```

Your action code is now able to communicate with the snips platform through the MQTT broker.

### Method 2 - Run your action code in the container

- Modify the `DockerFile` to include your action code.
- Modify the `snips-entrypoint.sh` to launch your services.
- Rebuild the container.
