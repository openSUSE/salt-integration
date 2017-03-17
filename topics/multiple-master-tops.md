- Topic: Multiple Master Tops
- Start Date: 2017-March-15

# Summary
[summary]: #summary

Tops aren't merged in Salt. Either one wins (first served).

# Motivation
[motivation]: #motivation

Two problems:

- Unable to merge multiple `top.sls`
- Unable to merge multiple outputs from the master tops.

# Details
[details]: #details

The main issue is that output from the custom mater tops is not
merged. Consider a script `foo.py` that returns tops,
something like this:

```python
def top(**kwargs):
    return {'base': ['foo']}
```

Then if you have another script `bar.py` that returns something like:

```python
def top(**kw):
    return {'base': ['bar']}
```

Then you place the following configuration in your master config:

```yaml
master_tops:
  foo: True
  bar: True
```

Then Master will use either one, but will not _merge_ the data into
`base`! The expectation is that the `base` should have both `bar` and
`foo` merged together.

This will allow to have one common configuration for multiple
independent users/environments.

**Workaround**

It is possible to make a merger plugin, which will read plug-plugins
for master tops and then return the output to the central Salt,
something like:

```
intergalactic_tops--- Returns {'foo': ['hello', 'world']}
  |
  |
  +--- Returns {'foo': ['hello']}
  |
  +--- Returns {'bar': ['world']}
```

In this workaround it is enough to specify:

```
master_tops:
  intergalactic_tops: True
```

However, in this case `intergalactic_tops` becomes a part of Salt
configuration. Which is not any good.

# Solution

PR upstream: https://github.com/saltstack/salt/pull/40113

# Drawbacks
[drawbacks]: #drawbacks

This can expose bugs, where leftovers were always overridden and never
caused a problems, while now they will be merged to the entire Tops.
