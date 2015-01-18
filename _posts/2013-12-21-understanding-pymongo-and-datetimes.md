---
layout: post
title:  "Understanding pymongo and datetime"
subtitle: "Date and time handling in Python is a minefield at the best of times. Dealing with MongoDB makes it worse."
date:   2013-12-21 18:26:29
categories: python
tags: python mongo encoding datetime pymongo
header-img: "img/numerals.jpg"
comments: true
---

The Python standard library's handling of dates and times is notoriously awful. By default, timezones are largely ignored and doing any kind of semi-useful operation on a `datetime` object is unintuitive and time-consuming (just try converting a naive `datetime` to a timestamp).

If you're using Mongo as your datastore, you'll probably already have some awareness that Mongo exposes BSON ISO dates as Python `datetime` objects. However, the details of how this works are somewhat non-obvious, especially when you consider naive vs timezone-aware `datetime`s.

So I decided to dig in and document what actually happens much for my own benefit as everyone else's.

# So, what happens?

Pymongo uses BSON to encode / decode documents which it saves / retrieves from the underlying data store.

BSON applies the following rules when it encounters a field which is a Python date time:

# Writing

Internally BSON encodes a `datetime` field as a unix timestamp - i.e. milliseconds since the Epoch. In order to do this it uses the following code:

```python
if isinstance(value, datetime.datetime):
    if value.utcoffset() is not None:
        value = value - value.utcoffset()
    millis = int(calendar.timegm(value.timetuple()) * 1000 +
                 value.microsecond / 1000)
    return BSONDAT + name + struct.pack("<q", millis)
```

So in a nutshell:

   * If the given `datetime` is timezone-aware, BSON will apply the UTC offset for that timezone to the `datetime` object so that when it is converted to an Epoch timestamp it is using the equivalent UTC time.
   * If the given `datetime` is naive, __BSON assumes the `datetime` is in UTC and directly converts it to an Epoch timestamp__. This means you need to be vary careful about saving naive `datetime`s which are not in UTC.

# Reading

Given that a BSON date is just an Epoch timestamp, BSON (and to some extent Mongo) then simply needs to create a `datetime` from that timestamp when it encounters such a field in a loaded document.

This is the code it uses to do this (note that this only applies as of pymongo 1.7):

```python
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
```

So in a nutshell:

   * If `tz_aware=True` is specified (more on this below), the returned `datetime` is created as timezone-aware and is set up with UTC as the time zone.
   * If `tz_aware=False` is specified, then you returned a naive `datetime`.

By default, pymongo will pass `tz_aware=False` to BSON when asking it to decode documents, so __your loaded documents will contain naive `datetime` objects__. To get PyMongo to return timezone-aware `datetime` objects, you’ll need to initialize your `Connection` or `MongoClient` object with `tz_aware=True`.

# Other thoughts

Pymongo/BSON’s approach to `datetime` objects as outlined above, though respecting timezone information (by applying the UTC offset), essentially discards that timezone information when saving to the database. That means that if retaining the timezone of a given `datetime` is important to you, you’ll need to store it separately and re-apply it to the `datetime` when it is loaded back out of the database (check out the excellent [Arrow](https://github.com/crsmithdev/arrow) library to make this super simple).

# References

* [Pymongo 1.7 FAQ - What is the correct way to handle timezones with PyMongo?](http://api.mongodb.org/python/1.7/faq.html#what-is-the-correct-way-to-handle-time-zones-with-pymongo)


<br />
