---
layout: post
title: Sending Voice Commands from Wear to RaspberryPi
date:
share: y
disqus: y
---
[jp]:https://ro.linkedin.com/in/ioanpaulpirau
[vali]:https://ro.linkedin.com/pub/valentin-mone/2/386/714
[hackcj]:http://clujinnovationdays.com/hackcj

>Or, the hackathon that never was.
>
>

In March there was supposed to be a hackathon called [HackCJ][hackcj] which seems to be coming soon ever since. I mentioned this hackathon to one of my long time friends - [Ioan-Paul][jp] - and he was excited about the prospect. So we proceeded to think about what we could do in the given timeframe that could also pass as a cool project with winning potential. He proposed that we would try to control some relays with a RaspberryPi in order to turn on and off various home appliances and I came up with the addition to do that by voice commands. Thus the idea of using a smart watch to send voice commands to a RaspberryPi was born. The only thing we needed was to complete our team so we called upon our other common long time friends, [Valentin][vali] and Florin, and when they accepted the challenge our team was born. So we proceeded to meet and discuss our hacking plan to make sure we get a fair chance at the prize, however just a couple of days before the hackathon date we incidentally found out that the event is postponed. Indefinitely.
<br/><br/>
Ah well, shit happens we said and we decided to move on and still try out our idea. So we set out some time to meet and implement a basic system that could send voice commands from Android Wear to a RaspberryPi. In the following paragraphs I will attempt to describe some details of what we have done, so grab your coffee and prepare to read some code - this is going to get technical.
<br/><br/>

{% highlight java %}
	private void displaySpeechRecognizer() {
        Intent intent = new Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH);
        intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL,
                RecognizerIntent.LANGUAGE_MODEL_FREE_FORM);
        startActivityForResult(intent, SPEECH_REQUEST_CODE);
    }
{% endhighlight %}