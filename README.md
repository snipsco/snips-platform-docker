
clone the repository
```
git clone https://github.com/snipsco/snips-platform-docker.git
```

enter the snips-platform-docker directory and the image folder you want to build , then build the image: 
```
docker build --no-cache -t snips-pulseaudio-docker .

```

result is `Successfully tagged snips-pulseaudio-docker:latest`


install pulesaudio on your host system or any network available host, you need to load the `module-native-protocol-tcp` module.
On a mac you can start pulseaudio like this:

macos install pulseaudio

```
brew install pulseaudio
```

mac os start pulseaudio

```
pulseaudio --load=module-native-protocol-tcp --exit-idle-time=-1 --daemon
```


find the ip adress of the host running the pulseaudio server , in the example below the address is `192.168.1.100` , you must change this to match your setting
in the example below we share the `~/.config/pulse` path form the host running pulseaudio to allow the docker image to connect , you can use other methode

start the docker image


```
docker run -it -e PULSE_SERVER=192.168.1.100 -e SNIPS_AUDIO_SERVER_ARGS="--alsa_capture pulse --alsa_playback pulse -v" -e SNIPS_AUDIO_SERVER_ENABLED="true" -v ~/.config/pulse:/var/empty/.config/pulse -v ~/.config/pulse:/root/.config/pulse --rm snips-pulseaudio-docker:latest
```

you can then connect to the running container and start `snips-watch` to check if everything is working as expected

find your snips container Id and start bash
```
docker ps -a
docker exec -it <ID> bash
```
