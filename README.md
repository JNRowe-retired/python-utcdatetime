I work all day with UTC datetimes in Python and I find some baffling things:

- Why do I need to import `pytz` just to get a `UTC` timezone class?
- Why does `datetime.utcnow()` return a *native* datetime with no UTC timezone?!
- Why does `datetime.isoformat()` use the `+00:00` format rather than `Z` for
  a UTC datetime?
- Why do I have to keep writing `%Y-%m-%dT%H:%M:%SZ` every time I parse or
  format a UTC datetime?
- Why do I have to handle all these timezone syntaxes: `Z`, `±HH:MM`, `±HHMM`,
  `±HH` ?!

This (opinionated) library gives you a UTC datetime with a sensible formatter
and a parser which handles sane ISO 8601 datetimes.

# Construct a UTC datetime

```python
>>> import utcdatetime
>>> my_utcdatetime = utcdatetime.utcdatetime(2011, 6, 1, 16, 45)
```

standard library equivalent:

```python
>>> import datetime
>>> import pytz
>>> my_datetime = datetime.datetime(2011, 6, 1, 16, 45, tzinfo=pytz.UTC)
```

# Format in Z-style ISO 8601

```python
>>> str(my_utcdatetime)
'2011-06-01T16:45:00Z'
```


standard library equivalent:
```python
>>> my_datetime.strftime('%Y-%m-%dT%H:%M:%SZ')
'2011-06-01T16:45:00Z'
```

# Parse from ISO 8601 Z, ±HH, ±HHMM, ±HH:MM style string

```python

>>> import utcdatetime
>>> utcdatetime.utcdatetime.from_string('2011-06-01T16:45:00Z')
utcdatetime.utcdatetime(2011, 6, 1, 16, 45)

>>> utcdatetime.utcdatetime.from_string('2011-06-01T16:45:00+00:00')
utcdatetime.utcdatetime(2011, 6, 1, 16, 45)

>>> utcdatetime.utcdatetime.from_string('2011-06-01T18:45:00+02:00')
utcdatetime.utcdatetime(2011, 6, 1, 16, 45)
```

Note that non-UTC timezones are automatically converted to UTC.

Doing this in the standard library is really nasty for a few reasons:

Suppose you use `datetime.datetime.strptime(my_string, '%Y-%m-%dT%H:%M:%S%z')`

- The `%z` argument to strptime only accepts `±HHMM`, not `±HH:MM` or `±HH`
- The `%z` argument also doesn't know about `Z`.
- The datetime you get back then needs to be converted using `.astimezone(UTC)`
- ... but you don't have a `UTC` class so you'll need to use `pytz` !
- And what about microseconds?

So you'll have to work out the format first, then run a different strptime
accordingly.

# Get now() and utcnow() that actually have a timezone

```python
>>> import utcdatetime
>>> utc_now = utcdatetime.utcdatetime.now()
>>> utc_now
utcdatetime.utcdatetime(2011, 1, 5, 18, 45, 36)

>>> str(utc_now)
`2011-01-05T18:45:36Z'
```

# Convert a datetime into a utcdatetime

Not a problem, as long as it's got a timezone:

```python
>>> import datetime
>>> import utcdatetime

>>> my_datetime = datetime.datetime(2010, 6, 10, 18, 45,
                                    tzinfo=pytz.timezone('Europe/London'))
>>> utcdatetime.utcdatetime.from_datetime(my_datetime)
utcdatetime.utcdatetime(2010, 6, 10, 17, 45)  # Note British Summer Time -> UTC
```
