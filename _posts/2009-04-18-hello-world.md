---
layout: post
title: The case for case statements
---

Your rails app that serves up image files needs to set the "Content-type"
in the header to match the files. Since we are only using jpegs, this is very easy.

{% highlight ruby %}
  # Takes a path to a jpeg
  def fetch(filename)
    send_file filename, :type => 'image/jpeg'
  end
{% endhighlight %}

But now you are serving up gifs also. No problem, just update the fetch method.

{% highlight ruby %}
  # Takes a path to a gif or jpeg
  def fetch(filename)
    ext = File.extname(filename)[1..-1]

    if ext == 'gif'
      content_type = 'image/gif'
    else
      content_type = 'image/jpeg'
    end
    
    send_file filename, :type => content_type
  end
{% endhighlight %}

That if else statement looks way too wordy, how about ternary style.

{% highlight ruby %}
  # Takes a path to a gif or jpeg
  def fetch(filename)
    ext = File.extname(filename)[1..-1]
    content_type = (ext == 'gif') ? 'image/gif' : 'image/jpeg'
    
    send_file filename, :type => content_type
  end
{% endhighlight %}

So, if the file is a gif, set the content type to 'image/gif', otherwise
default to 'image/jpeg'. This is alright only because we know we aren't serving up
any other file types. Or at least that's what we tell ourselves isn't it.

Are you disgusted with this code already? Good. I am too.

Can you guess what happens next? Of course, now we need to support another
file type, PNGs. No problem! We'll have to switch the logic though...and while
we're at it, let's raise an exception if the extension of the file isn't one that
we expect.

{% highlight ruby %}
  # Takes a path to a gif, jpeg or png
  def fetch(filename)
    ext = File.extname(filename)[1..-1]
    
    if ext == 'gif'
      content_type = 'image/gif'
    elsif ext == 'jpg' || ext == 'jpeg'
      content_type = 'image/jpeg'
    elsif ext == 'png'
      content_type = 'image/png'
    else
      raise "This file type is unsupported"
    end
    
    send_file filename, :type => content_type
  end
{% endhighlight %}

While this isn't terrible, it's staring to look messy. A lazy coder would leave
it like this, and would probably continue to copy and paste elsif statements
downward if we needed to support more file types. What we really need to do here
is change this to a case statement.

{% highlight ruby %}
  # Takes a path to a gif, jpeg or png
  def fetch(filename)
    ext = File.extname(filename)[1..-1]
    
    case ext
    when 'gif'
      content_type = 'image/gif'
    when 'jpg', 'jpeg'
      content_type = 'image/jpeg'
    when 'png'
      content_type = 'image/png'
    else
      raise "This file type is unsupported"
    end
    
    send_file filename, :type => content_type
  end
{% endhighlight %}

This is definitely more readable. We can do better though.

{% highlight ruby %}
  # Takes a path to a gif, jpeg or png
  def fetch(filename)
    ext = File.extname(filename)[1..-1]
    
    content_type = case ext
    when 'gif'
      'image/gif'
    when 'jpg', 'jpeg'
      'image/jpeg'
    when 'png'
      'image/png'
    else
      raise "This file type is unsupported"
    end
    
    send_file filename, :type => content_type
  end
{% endhighlight %}

Wait, we can do better than that.

{% highlight ruby %}
  # Takes a path to a gif, jpeg or png
  def fetch(filename)
    ext = File.extname(filename)[1..-1]
    
    content_type = case ext
    when 'gif'         : 'image/gif'
    when 'jpg', 'jpeg' : 'image/jpeg'
    when 'png'         : 'image/png'
    else raise "This file type is unsupported"
    end
    
    send_file filename, :type => content_type
  end
{% endhighlight %}

This is looking much nicer. At this point or earlier you might have realized
that you really need this extension logic in a separate method. Let's see how
that looks.

{% highlight ruby %}
  # Takes a path to a gif, jpeg or png
  def fetch(filename)
    send_file filename, :type => content_type(filename)
  end
  
  def content_type(filename)
    case File.extname(filename)[1..-1]
    when 'gif'         : 'image/gif'
    when 'jpg', 'jpeg' : 'image/jpeg'
    when 'png'         : 'image/png'
    else raise "This file type is unsupported"
    end
  end
{% endhighlight %}

As an added bonus we were able to remove the variable assignment since the
method will return the value of the last statement, in the case the case statement.

Let's pretend it's been a few months, and now we've
realized we need support for not just images but video files.

{% highlight ruby %}
  # Takes images and video files
  def fetch(filename)
    send_file filename, :type => content_type(filename)
  end
  
  def content_type(filename)
    case File.extname(filename)[1..-1]
    when 'gif'         : 'image/gif'
    when 'jpg', 'jpeg' : 'image/jpeg'
    when 'png'         : 'image/png'
    when 'mov'         : 'video/quicktime'
    when 'mpg', 'mpeg' : 'video/mpeg'
    when 'avi'         : 'video/x-msvideo'
    else raise "This file type is unsupported"
    end
  end
{% endhighlight %}

Not too bad. But now imagine we want to add support for a whole host of other
files. Let's say this will add 100 lines to this case statement. We have outgrown
our case statement at this point. We need a hash.

{% highlight ruby %}
  # Takes any file
  def fetch(filename)
    send_file filename, :type => content_type(filename)
  end
  
  def content_type(filename)
    ext  = File.extname(filename)[1..-1]
    content_hash[ext] || raise("This file type is unsupported")
  end
  
  def content_hash
    {
      'gif'  => 'image/gif',
      'jpg'  => 'image/jpeg',
      'jpeg' => 'image/jpeg',
      'png'  => 'image/png',
      'mov'  => 'video/quicktime',
      'mpg'  => 'video/mpeg',
      'mpeg' => 'video/mpeg',
      'avi'  => 'video/x-msvideo'
    }
  end
{% endhighlight %}

We've gone from having our extension to content type mappings imbedded in a
control structure (the case statement), to storing the extension to content type
mappings in a data structure ( a Hash ).

This opens up some opportunities for us. If our list of content types grew to the
hundreds, we could move them into a csv file or yaml file. This file would
be loaded into a Hash.
