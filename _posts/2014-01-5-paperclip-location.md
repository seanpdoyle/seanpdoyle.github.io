---
layout: post
title: Paperclip Location
description: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
modified: 2013-05-31
tags: [ruby rails paperclip geolocation]
comments: true
share: true
---

Pulling location data from photos has never been easier.
Extracts GeoLocation data from an image during Paperclip processing
and attaches it to the associated model.

## Installation

Add this line to your application's Gemfile:

```ruby
  gem 'paperclip-location'
```

And then execute:

```console
$ bundle
```

Or install it yourself as:

```console
$ gem install paperclip-location
```

## Usage

Use it like any other `Paperclip::Processor`

```ruby
class PlaceOfInterest < ActiveRecord::Base

  has_attached_file :photo, styles: { large: '600x600#' },
                    processors: [:thumbnail, :location]

end
```

The processor expects that the model in question has the following:

* `location_locked` - a boolean flag to determine if the location has been manually overridden. If true, the file will be unprocessed.
* `lat` - a float representing the latitude
* `lng` - a float representing the longitude

If you don't have either, run a migration to add them

```ruby
class AddLocationToModel < ActiveRecord::Migration
  def self.change
    add_column :model, :location_locked, :boolean, default: false, null: false
    add_column :model, :lat, :float
    add_column :model, :lng, :float
  end
end
```