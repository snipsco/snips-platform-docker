
clone the repository
```
git clone https://github.com/snipsco/snips-platform-docker.git
```

enter the 
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


start the docker image

```
docker run -it -e PULSE_SERVER=192.168.1.100 -e SNIPS_AUDIO_SERVER_ARGS="--alsa_capture pulse --alsa_playback pulse -v" -e SNIPS_AUDIO_SERVER_ENABLED="true" -v ~/.config/pulse:/var/empty/.config/pulse -v ~/.config/pulse:/root/.config/pulse --rm snips-pulseaudio-docker:latest
```

```
docker run -it -e PULSE_SERVER=192.168.1.100 -e SNIPS_AUDIO_SERVER_ARGS="--alsa_capture pulse --alsa_playback pulse -v" -e SNIPS_AUDIO_SERVER_ENABLED="true" -v ~/.config/pulse:/var/empty/.config/pulse -v ~/.config/pulse:/root/.config/pulse --rm snips-pulseaudio-docker:latest
```
