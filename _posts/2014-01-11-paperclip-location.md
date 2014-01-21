---
layout: post
title: Paperclip Location
modified: 2014-01-11
---

Pulling geo-location from photos with Rails has never been easier.

[Paperclip](https://github.com/thoughtbot/paperclip) is a file attachment gem from the good folks at [Thoughtbot](https://github.com/thoughtbot). [Paperclip-Location](https://github.com/seanpdoyle/paperclip-location) is a `Paperclip::Processor` that extracts an image's `latitude` and `longitude` from `EXIF` metadata.

## The Setup

The processor requires works with `ActiveRecord` models, and requires 3 fields:

* `lat` - a decimal representing the latitude
* `lng` - a decimal representing the longitude
* `location_locked` - a boolean flag to determine if the location has been manually set. During image processing, if the `location_locked` field is true, the record's current location will remain and `EXIF` processing will be skipped.

First, add them to the table with a migration.

<script src="https://gist.github.com/seanpdoyle/8294526.js?file=20140110012324_create_places.rb"></script>

Then, include `:location` in the array of [`Paperclip::Processors`](https://github.com/thoughtbot/paperclip#custom-attachment-processors) within the `has_attached_file` macro.

Finally, `paperclip-location` expects that processed files are images with `EXIF` metadata. To require this, add a `Paperclip` validation on `:content_type`.

<script src="https://gist.github.com/seanpdoyle/8294526.js?file=place.rb"></script>

## The Payoff

Now, whenever an image with `GPS` information in its `EXIF` headers is attached to a `Place`, its `lat` and `lng` fields will automatically be updated.


<script src="https://gist.github.com/seanpdoyle/8294526.js?file=location"></script>


### Further Reading

* View the source code on [GitHub](https://github.com/seanpdoyle/paperclip-location)
* A working code example can be found [here](https://gist.github.com/seanpdoyle/8294526).
* For a real example from the wild, checkout the source for [Chief](https://github.com/seanpdoyle/chief), an open-source skate spot catalog.