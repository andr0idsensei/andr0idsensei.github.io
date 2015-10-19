---
layout: post
title: Sending Voice Commands from Android Wear to RaspberryPi
date: 2015-10-18
share: y
disqus: y
---
[jp]:https://ro.linkedin.com/in/ioanpaulpirau
[vali]:https://ro.linkedin.com/pub/valentin-mone/2/386/714
[florin]:https://ro.linkedin.com/pub/florin-toma/9/229/b67

[wear]:https://www.android.com/wear
[pi]:https://www.raspberrypi.org
[iot]:https://en.wikipedia.org/wiki/Internet_of_Things
[grovepi]:http://www.dexterindustries.com/grovepi
[dexterind]:http://www.dexterindustries.com
[relay]:http://www.seeedstudio.com/wiki/Grove_-_SPDT_Relay(30A)


>Or, the hackathon that never was.
>
>

Lately the Startup fever hit my city and a bunch of events dedicated to entrepreneurship and innovation are spawning all over the place. One such event was the reason that me and a few friends decided to test out the possibility to send voice commands from an [Android Wear][wear] smartwatch to a [RaspberryPi][pi] computer in order to activate/deactivate some electrical devices such as fans or lightbulbs. The actual hackathon that we prepared for never happened, but we decided to implement our idea in order to see a working proof of concept anyway. Who knows, maybe we can come up with something in the future or participate at other events to show off.
<br/><br/>

Anyway the whole thing was a good experience from which I learned some stuff that I would like to share with others who might have the [IoT][iot] fever. Following, I'll describe a bit our solution and how we made things work.
<br/><br/>

So, here's the story behind the idea: we started from thinking how cool it would be to turn on or off your fans or other home appliances by using your smartwatch to voice command them. Normally home appliances are just plain electrical devices who need to be manually turned on or off and they rely on electrical circuits from your home to deliver their power. At first look, you might be tempted to think: how can they be controlled from a smart watch? They are just ordinary home appliances, not even the smart kind you might find in stores lately, that are integrated with all kinds of home automation solutions and which, of course, have a price tag to match their "smarts". But where there's a will there's a way.
<br/><br/>

<em>Disclaimer:</em> The things I'll describe next won't make your home apliances smart, they're still gonna be the good old dumb appliances you've come to appreciate and love. What we did is just a simple trick to turn them on or off using your voice, no features change, the devices still remain as they are.
<br/><br/>

Before we go into the nitty gritty details of how everything fits together to achieve our goals, there's an inventory of hardware parts that we used in order to get this working. We used a RaspberryPi 2 computer - but any version compatible with the [GrovePi][grovepi] board from [Dexter Industries][dexterind] would do. Of course, used the GrovePi board, which is an extension to the GPIO on the RaspberryPi. The part that would control an electrial appliance would be an electrical relay. We used one of the 30 amps [SPDT relays][relay] from Dexter Industries which can be controlled with the GrovePi board. This relay would actually be connected to an electrical circuit that powers the appliance we want to control. We didn't do that part, for the purposes of our proof of concept, just clicking the relay on and off with the RaspberryPi was enough. The other important hardware pieces involved were an Android phone and an Android Wear smartwatch, specifically my Nexus 4 and LG G Watch R. Of, course for the whole thing to be connected, a local network would be necessary in which the phone and the Pi can access each other.
<br/><br/>

With the hardware inventory out of the way, I'll explain what we did in the code in order to prove this can work. The first step was to create and Android application with both mobile and wear modules. The part that would run on the phone would be responsible for receiving the voice command result from the wearable, processing the text into a simple <em>on</em> or <em>off</em> command and then passing it to the RaspberryPi over a TCP connection via a socket. The wear module would be responsible for getting the voice input from the default Speech Recognizer activity and passing that to the mobile module. This whole messaging part is done through the GoogleAPiClient, as recommended in the official documentation.
<br/><br/>

The second software part would be the RaspberryPi code which would listen on a TCP socket for the commands coming from the Android application. By the time they would get here, the commands would be simple <em>on/off</em> texts which would tell the Pi to turn on or off the attached relay. This part was achieved by using a Python script which acted as a server socket listening for <em>on/off</em> commands. When one of the commands would come, it would get passed to the GrovePi API and the relay would be turned on and off. The server script was integrated with the GrovePi API scripts in order for this to work.
<br/><br/>

Next, I'll add some code samples for the more important parts, to exemplify how the story would look in the code.
<br/><br/>

Displaying the Speech recognizer on the wearable (I used the code from the Google documentation): for this you just need to use <code>startActivityForResult</code> with the intent and code described below.
{% highlight java %}
	private void displaySpeechRecognizer() {
        Intent intent = new Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH);
        intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL,
                RecognizerIntent.LANGUAGE_MODEL_FREE_FORM);
        startActivityForResult(intent, SPEECH_REQUEST_CODE);
}
{% endhighlight %}
<br/><br>

This would be the async task used for sending the voice input to the mobile phone. It should be started in the <code>onConnected</code> method of your GoogleAPiClient.
{% highlight java %}
private class SendSpokenTextToNodesTask extends AsyncTask<String, Void, Void> {

        private Collection<String> getNodes() {
            HashSet<String> results = new HashSet<String>();
            NodeApi.GetConnectedNodesResult nodes =
                    Wearable.NodeApi.getConnectedNodes(mGoogleApiClient).await();
            for (Node node : nodes.getNodes()) {
                results.add(node.getId());
            }

            Log.d(TAG, "connected nodes:" + results);
            return results;
        }

        private void sendSpeechResultToNode(String speechToText, String nodeId) {
            Wearable.MessageApi.sendMessage(
                    mGoogleApiClient, nodeId, START_SPEECH_TO_TEXT_PROCESSOR, speechToText.getBytes()).setResultCallback(
                    new ResultCallback<MessageApi.SendMessageResult>() {
                        @Override
                        public void onResult(MessageApi.SendMessageResult sendMessageResult) {
                            if (!sendMessageResult.getStatus().isSuccess()) {
                                Log.e(TAG, "Failed to send message with status code: "
                                        + sendMessageResult.getStatus().getStatusCode());
                            }
                        }
                    }
            );
        }

        @Override
        protected Void doInBackground(String... params) {
            Log.d(TAG, "doInBackground...");
            for(String node : getNodes()) {
                Log.d(TAG, "sending " + mSpokenText + " to node " + node);
                sendSpeechResultToNode(mSpokenText, node);
            }

            return null;
        }
}
{% endhighlight %}
<br/><br>

Make sure you initialize the GoogleAPIClient first. This can be done when your activity is created. The <code>connect()</code> method of the GoogleApiClient instance should be called when the Speech Recognizer returns it's result, namely in the <code>onActivityResult</code> method of the calling activity.
{% highlight java %}
private void initApiClient() {
        mGoogleApiClient = new GoogleApiClient.Builder(this)
                .addConnectionCallbacks(new GoogleApiClient.ConnectionCallbacks() {
                    @Override
                    public void onConnected(Bundle connectionHint) {
                        Log.d(TAG, "onConnected: " + connectionHint);
                        new SendSpokenTextToNodesTask().execute(mSpokenText);

                    }
                    @Override
                    public void onConnectionSuspended(int cause) {
                        Log.d(TAG, "onConnectionSuspended: " + cause);
                    }
                })
                .addOnConnectionFailedListener(new GoogleApiClient.OnConnectionFailedListener() {
                    @Override
                    public void onConnectionFailed(ConnectionResult result) {
                        Log.d(TAG, "onConnectionFailed: " + result);
                    }
                })
                .addApi(Wearable.API)
                .build();
}
{% endhighlight %}
<br/><br>

On the mobile module side, I created a simple activity with 2 edit texts and 2 buttons in order to be able to set or clear the TCP host and port of the RaspberryPi server. This part is not very interesting since it just saves the host address and the port in a SharedPreference file. The interesting bit on the mobile side would be the service listening to the wear messages. This is also responsible for opening the TCP connection to the server and sending the simplified <em>on/off</em> commands. I'll add the whole thing below:
{% highlight java %}
public class VoiceCommandListenerService extends WearableListenerService {
    /**
     * The message event path.
     */
    private static final String START_SPEECH_TO_TEXT_PROCESSOR = "/start/SpeechToTextProcessor";

    /**
     * The Log tag.
     */
    private static final String TAG = "VoiceCommandListener";

    @Override
    public void onMessageReceived(MessageEvent messageEvent) {
        if (messageEvent.getPath().equals(START_SPEECH_TO_TEXT_PROCESSOR)) {

            String spokenText = new String(messageEvent.getData());
            Log.d(TAG, "received spoken text: " + spokenText);

            SharedPreferences prefs = getApplicationContext().getSharedPreferences(PiBrainActivity.PI_BRAIN_PREFS, Context.MODE_PRIVATE);
            String host = prefs.getString(PiBrainActivity.HOST, "raspberrypi");
            String port = prefs.getString(PiBrainActivity.PORT, "3333");
            Log.d(TAG, "host: " + host + " port " + port);

            SendPiCommand task = new SendPiCommand();
            task.execute(host, port, spokenText);
        }
    }

    /**
     * Async task to send commands to the Raspberry Pi.
     */
    private static class SendPiCommand extends AsyncTask<String, Void, Void> {

        @Override
        protected Void doInBackground(String... params) {
            String host = params[0];
            String spokenText = params[2];
            int port = Integer.parseInt(params[1]);

            Socket commandSocket = null;
            String command = CommandFilter.filterCommand(spokenText).getCommand();
            Log.d(TAG, "the Pi command: " + command);

            try {
                commandSocket = new Socket(host, port);
                PrintWriter out = new PrintWriter(commandSocket.getOutputStream(), true);
                out.write(command);
                out.flush();

            } catch (IOException e) {
                e.printStackTrace();
            }

            return null;
        }
    }
}
{% endhighlight %}
<br/><br>

On the RaspberryPi side, here is the server listening script. This also is an adaption from some server socket examples in Python we found on their official documentation site.
{% highlight python %}
import SocketServer
import socket

#imports for PiBrainRelay
import time
import grovepi

class PiBrainRelay:
	"""
	The Relay actuator class
	"""
	
	#relay = 4 #relay will be connected to the digital port 4
	
	def __init__(self): #constructor and class init
		grovepi.pinMode(4, "OUTPUT")
	
	def actuate(self, switch = 0):
		if switch == 0:
			grovepi.digitalWrite(4, 0)
			print "relay is off"
		else:
			grovepi.digitalWrite(4, 1)
			print "relay is on"
			
			
class PiBrainServer(SocketServer.BaseRequestHandler):
    """
    The RequestHandler class for our server.

    It is instantiated once per connection to the server, and must
    override the handle() method to implement communication to the
    client.
    """

    def handle(self):
		try:
		    theRelay = PiBrainRelay()
			# self.request is the TCP socket connected to the client
			self.data = self.request.recv(1024).strip()
			print "{} wrote:".format(self.client_address[0])
			print self.data
			#act on the relay
			if self.data == "on":
				theRelay.actuate(1)
			elif self.data == "off":
				theRelay.actuate(0)
			else:
				print "command not recognized..."
			# just send back the same data, but upper-cased
			self.request.sendall(self.data.upper())
		except KeyboardInterrupt:
			theRelay.actuate(theRelay, 0)
		except IOError:
			print "IOError"				
		

if __name__ == "__main__":
    print socket.gethostname()
    HOST, PORT = "192.168.1.9", 3333

    # Create the server, binding to localhost on port 9999
    server = SocketServer.TCPServer((HOST, PORT), PiBrainServer)
	
    # Activate the server; this will keep running until you
    # interrupt the program with Ctrl-C
    server.serve_forever()

{% endhighlight %}
<br/><br>

The last bit of code is the GrovePi methods for <code>digitalWrite</code>, which sends the on/off signal to one of the digital pins on the GrovePi board, and <code>pinMode</code>, which sets the pin you want to connect the relay to on the board. These use the GrovePi library calls which are available by installing the GrovePi board on your RaspberryPi.
{% highlight python %}
def digitalWrite(pin, value):
	write_i2c_block(address, dWrite_cmd + [pin, value, unused])
	return 1

def pinMode(pin, mode):
	if mode == "OUTPUT":
		write_i2c_block(address, pMode_cmd + [pin, 1, unused])
	elif mode == "INPUT":
		write_i2c_block(address, pMode_cmd + [pin, 0, unused])
	return 1
{% endhighlight %}
<br/><br>

This is it! I hope it wasn't too boring even if it is a long post. If you like it, if you don't, if you have questions, please let me know in the comments below. Before we part, I would like to give the proper credits and thanks to my friends [Ioan Paul][jp], [Valentin][vali] and [Florin][florin] who were also involved in this project in various degrees. 'Till next time, have fun!