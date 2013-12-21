---
layout: post
title:  "Understanding pymongo and datetime"
date:   2013-12-21 18:26:29
categories: pymongo datetime python mongo encoding
---

The Python standard library's handling of dates and times is notoriously awful. By default, timezone's are largely ignored and doing any kind of semi-useful operation on a `datetime` object is unintuitive and time-consuming (just try converting a naive datetime to a timestamp).

If you're using Mongo as your datastore, you'll probably already have some awareness that Mongo exposes BSON ISO dates as Python datetime objects. However, the rules that govern the details of how this works are somewhat non-obvious, especially when you consider naive vs timezone-aware `datetime`s.

So I decided to dig in and document what actually happens much for my own benefit as everyone else's.

# So, what happens?

PyMongo uses BSON to encode / decode documents which it saves / retrieves from the underlying data store.

BSON applies the following rules when it encounters a field which is a Python date time:

# Writing

Internally BSON encodes a `datetime` field as a unix timestamp - i.e. milliseconds since the Epoch. In order to do this it uses the following code:
{% highlight python %}
    if isinstance(value, datetime.datetime):
        if value.utcoffset() is not None:
            value = value - value.utcoffset()
        millis = int(calendar.timegm(value.timetuple()) * 1000 +
                     value.microsecond / 1000)
        return BSONDAT + name + struct.pack("<q", millis)
{% endhighlight %}
So in a nutshell:

   * If the given `datetime` is timezone-aware, BSON will apply the UTC offset for that timezone to the `datetime` object so that when it is converted to an Epoch timestamp it is using the equivalent UTC time.
   * If the given timezone is naive, BSON basically assumes the `datetime` is in UTC and directly converts it to an Epoch timestamp.



# Reading

Given that a BSON date is just an Epoch timestamp, BSON (and to some extent Mongo) then simply needs to create a `datetime` from that timestamp when it encounters such a fields in a loaded document.

This is the code it uses to do this:

{% highlight python %}
def _get_date(data, position, as_class, tz_aware, uuid_subtype):
    millis = struct.unpack("<q", data[position:position + 8])[0]
    diff = millis % 1000
    seconds = (millis - diff) / 1000
    position += 8
    if tz_aware:
        dt = EPOCH_AWARE + datetime.timedelta(seconds=seconds)
    else:
        dt = EPOCH_NAIVE + datetime.timedelta(seconds=seconds)
    return dt.replace(microsecond=diff * 1000), position
{% endhighlight %}

So in a nutshell:

   * If `tz_aware=True` is specified (more on this below), the returned `datetime` is created as timezone-aware and is set up with UTC as the time zone.
   * If `tz_aware=False` is specified, then you returned a naive `datetime`.

By default, PyMongo will pass `tz_aware=False` to BSON when asking it to decode documents, so your loaded documents will contain naive `datetime` objects. To get PyMongo to return timezone-aware `datetime` objects, you’ll need to initialize your Connection or MongoClient object with `tz_aware=True`.

# Other thoughts

PyMongo/BSON’s approach to `datetime` objects as outlined above, though respecting timezone information (by applying the UTC offset), essentially discards that timezone information when saving to the database. That means that if retaining the timezone of a given `datetime` is important to you, you’ll need to store it separately and re-apply it to the `datetime` when it is loaded back out of the database (check out the excellent Arrow library to make this super simple).

