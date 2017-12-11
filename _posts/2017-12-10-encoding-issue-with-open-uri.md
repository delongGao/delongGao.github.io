---
layout: default
title:  "Encoding Issue With open-uri"
date:   2017-12-10 15:07:01 -0600
categories: working_journals ruby encoding open-uri
---
## Intro
In my recent project, I have encountered a bug that is related to UTF-8 characters not being encoded correctly for some text files. But what makes this hard to debug is it only happens on production. Here are some summary and documentation about this issue and the solution I found.

## Symptoms
- When these text files are processed on my local dev environment, everything works fine, even for UTF-8 characters, like __`__.
- When the same file is processed on production, I start to see errors show up, specifically `Encoding::UndefinedConversionError: "\xE2" from ASCII-8BIT to UTF-8`.
- Normal text files with no special UTF-8 characters works fine on production.

## Findings
With some search on the web, apparently this error usually got thrown when you tell ruby to: 
1. Write to a file with a specific encoding
2. The content you want to write is not totally compatible with the encoding you want

In this case, it turns out the content I want to write into a tempfile contains an apostrophe, but when the content is prepared, it's not correctly encoded, as a result the apostrophe is encoded as binary format. Then later when I was trying to write it into the tempfile, ruby is pissed and complains that she doesn't know how to deal with this binary format character.

## Solutions
Now things all make sense, we can think of some possible solutions:

### 1. Telling Ruby to be tolerant at writing time
Some people suggested this, basically when you open a file with `file.open` in ruby, you can add `b` as part of the mode to tell ruby to also allow binary format, as follow:
```ruby
File.open(file_path, 'wb') { |f| f.write(my_content) }
```
Nothing wrong about this approach, and it's a legitimate use of the open mode, but my problem is that I don't think this is solving the root cause of the problem: 

**Instead of making the content encoded correctly, we are telling ruby to accept an abnormal encoding.**

### 2. Get the encoding right when prepare the content
When tracking back to find where the content is generated, I found out it was read in from a S3 url using the ruby stdlib `open-uri` like this:
```ruby
raw_content = open(input_url, &:read)
```

I decided to add a breakpoint here to inspect the content at this moment. As I was expecting, the `raw_content` does contain the evil character `\xE2`! So what's wrong with `open` and `read`?

With some further search, I found the reason from this [StackOverflow post](https://stackoverflow.com/questions/24936452/encodingundefinedconversionerror-when-using-open-uri): basically when use `open` to read a file (or reading file in general), you have to tell ruby what's the current encoding, otherwise it will use the default one, which is `ASCII-8BIT`. This matches the error we saw at the beginning. Ruby was trying to tell me, the content I was trying to write was beyond her default guessing!

And from the same SO post, I learnt that you can check the default external encoding of your environment by doing this: `Encoding.default_external`. And with some further digging in the `open-uri` documentation, I found out you can set the encoding of the io stream. So here comes my final solution:

```ruby
result = open('some-page-uri') do |io|
  io.set_encoding(Encoding.default_external)
  io.read
end
```
I like this solution because it's targeting the root cause: the content you are trying to write should be encoded as you expect.