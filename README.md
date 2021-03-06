# kaldi.js

This is a simple webpage that hooks into a kaldi-gstreamer-server. The main code is in
js/audio.js. That is where we register to receive audio, send the audio to the server, and receive
translations back.

# Backend requirements

## kaldi

Must have kaldi installed in order to run the servers.

Follow install instructions at ```https://github.com/kaldi-asr/kaldi```

*Note: kaldi takes a long time to compile.*

## gstreamer server

This is a python server that uses kaldi to translate audio it receives over websockets.

Install instructions at ```https://github.com/alumae/kaldi-gstreamer-server```

## gstreamer online neural net decoder plugin

According to [this section](https://github.com/alumae/kaldi-gstreamer-server#using-the-kaldinnet2onlinedecoder-based-worker), in order to use the neural net decoder with the gstreamer server, you have to compile a special plugin for streamer.

Repo found at ```https://github.com/alumae/gst-kaldi-nnet2-online```

# Running the backend

I used this bash script to run all the needed things for the server:

```bash
#!/bin/bash

rm *.out

# run the server
nohup python kaldigstserver/master_server.py --port=8888 > server.out &

# run a decoding worker
# **NOTE**: you'll have to fill out the path in the export on the next line
export GST_PLUGIN_PATH=<path to gst-kaldi-nnet2-online>/src
nohup python kaldigstserver/worker.py -u ws://localhost:8888/worker/ws/speech -c sample_english_nnet2.yaml > worker.out &
```

This will run the server & 1 worker, which is what the server hands off the audio to to do the speech recognition.

To stop everything you can run (assuming there aren't any other python processes running):
```bash
> killall python
```

# Running the frontend

After the backend is running, you can launch the following python command to host an http server in this directory:

```bash
python -m SimpleHTTPServer
```

Then navigate to ```localhost:8000``` (or ```<computer name>:8000```) and you should see the index.html content.

**Note**: Chrome will only ask for audio access if the webpage is loaded over an HTTPS connection, so for testing I had to use Firefox.
