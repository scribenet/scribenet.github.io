---
layout: post
title: Text Messaging Library
---

Recently, we wanted to be able to send text message alerts to a group of individuals as an early-warning method to identify degradation or failure of our systems. While there are an [array](http://www.twilio.com/sms) of [messaging](https://carouselsms.com/) [api](http://www.textmarks.com/front/dev/) [options](http://blog.mashape.com/post/56272188360/list-of-50-sms-apis), we settled on [Clockwork](http://www.clockworksms.com/). 

Unlike other solutions, Clockwork provides a simple SMS messaging API with no additional bells and whistles; we weren't looking for VOIP services, conferencing features, or anything else. Clockwork developed a clean messaging API, perfect for our needs.

Clockwork offers libraries for a handful of languages, including PHP, Ruby, Java, Python, and C#, though our interest was in their HTTP XML API. Using this, we were able to build a small and simple Symfony bundle to interface with Clockwork's XML API. Since they already had a PHP library that took advantage of the XML API, much of our code was was derived from their [PHP wrapper](https://github.com/mediaburst/clockwork-php).

### Configure the bundle

Our [Clockwork Bundle](https://github.com/scribenet/ScribeClockworkBundle) includes parameter injection via the Symfony service container. At a minimum, you must configure your API key.

```
scribe_clockwork:
    api_key: your-api-key-goes-here
```

A complete reference of the available configuration options can be found within the project's [README.md](https://github.com/scribenet/ScribeClockworkBundle#configuration).

### Sending a message

This bundle includes a single service, `scribe_clockwork`. To send a message, simply request the service and call the `send` method. You can optionally use the `sendMultiple` method to pass multiple phone numbers and messages.

{% highlight php startinline %}
// request the service
$cw = $this->get('scribe_clockwork');

// send a single message
$message_id = $cw->send('12223334444', 'Your text message');

// send multiple messages
$message_ids = $cw->sendMultiple([
    '12223334444' => 'Your text message',
    '23334445555' => 'Your second message'
]);
{% endhighlight %}

### Additional Features

We provide methods for checking API key validity, available credit, and available balance. You can use these methods as follows:

{% highlight php startinline %}
// request the service
$cw = $this->get('scribe_clockwork');

// check api key validity
if ($cw->isValidApiKey() === false) {
	echo 'Invalid API Key';
}

// get available credits
$credits = $cw->getCredit();

// get available balance
list($balance, $symbol, $code) = $cw->getBalance();
{% endhighlight %}

### Exceptions

All the available public methods may throw `ClockworkException` exceptions. As such, it is advisable to wrap your API calls within a try/catch block to avoid exceptions bubbling up. In a production environment, our first example would be rewritten to look like this:

{% highlight php startinline %}
// request the service
$cw = $this->get('scribe_clockwork');

try {
	// send a single message
	$message_id = $cw->send('12223334444', 'Your text message');
} catch (ClockworkException $e) {
	// perform some action if exception occurs
}
{% endhighlight %}