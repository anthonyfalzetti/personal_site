---
title: "Server_is_running_out_of_space"
date: 2018-04-07T14:40:57-06:00
draft: false
---

One day while working the service desk a critical issue came across my desk. One of the servers we were running Ruby on Rails on was running out of disk space. This was a very common issue that we had a series of steps to address:
1. Ensure previous deploys of the app are compressed
2. Ensure that logrotate is enabled and the current log is a reasonable size
3. Ensure reports uploaded or exported are compressed and of reasonable size

After running through the usual steps I was left with a virtual machine that only had 4% space left and growing every second. Time to run some basic command line commands to find where the space is being used.

First I ran `du -ah /` to find all files on the machine and their sizes. I did this so I can see if any directories are showing up consistently. Luckily a single directory was showing up repeatedly so I switched to that directory and ran `du -sh` to see see how much the directory actually is taking up. Turns out it was taking up a significant amount of space. Before doing some basic cleanup on this directory I wanted to see if there was a pattern to how quickly this directory was growing.

Now the true problem: If nothing changes how long till we completely run out of space? I began by running

```bash
find . -maxdepth 1 -mtime -30 | wc -l
```

This command returned the total files that had been updated with the past 30 days, but didn't really tell me how much space we were consuming each day. Time to find my mentor to see if he has any tricks up his sleeves.

```bash
find . -type f -mtime -30 | xargs ls -aoltr | awk '{print $4, $5}' | ruby -ne 'BEGIN { @count = Hash.new(0) }; chomp; (file_size, date) = $_.split(/\s+/, 2); @count[date] += file_size.to_i; END { @count.each{|k,v| puts "#{k}: #{v}"}}' | sort
```

This returned a list of dates for the past 30 days with the amount of bites consumed each day. Now I can accurately report how long we have before we hit 0, and find any patterns as to when the consumption increased.

## Breaking down the solution

Rather than leave this to some magical script my mentor whipped up I broke it down into separate parts to learn how it solved my problem.

Starting at the directory list all the file names including relative file paths and pipe the result to the next function

```bash
find . -type f -mtime -30 |
```

List the files from the previous command in order oldest to newest and include date, size, owner, and privileges and pipe the result to the next function
```bash
xargs ls -aoltr
```

Take the ordered list of files and only take the size and date of the files and pipe the result to the next command. This works really well if the date is in a YYYYMMDD format or similar. If it is in the "OCT 14" format some changes can be made to improve this command.
```bash
awk '{print $4, $5}'
```

This runs ruby as a command line and takes the list of size, date files and essentially wraps them in a `while gets end` loop.
```bash
ruby -ne '..'
```

This creates a new hash, takes the list of list of size, and date files removes the white space except for in between the date, size. Using the date as the key aggregates the sizes of the files and prints to the console the date and total file sizes. Again piping the return value to the next command.
```ruby
BEGIN
  @count = Hash.new(0)
  chomp
  (file_size, date) = $_.split(/\s+/, 2)
  @count[date] += file_size.to_i
END
{ @count.each{|k,v| puts "#{k}: #{v}"}}
```

Finally taking the list of date: filesize formatted strings and sort it from oldest to newest (assuming the dates are in `YYYYMMDD` format)
```bash
sort
```

I am sure there are ways to improve this, or other methods and tricks to solve this problem, and I would like to learn them. So if you know of a way to improve this please drop a comment so we can have a discussion.