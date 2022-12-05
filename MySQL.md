---
title: "Netty"
date: 2022-01-21T12:40:52Z
draft: false
---

**Database**: a set of physics operation system file or other types file.

**Instance**: MySQL database is composed of backstage threads and a area of share memory.


## InnoDB

Backstage Threads:

1. Master Thread: flush data in buffer pool into disk.
2. IO Thread: process AIO's callback. include 4 write threads, 4 read threads, 1 insert buffer thread, adn 1 log IO thread.
3. Purge Thread: collect undo log.
4. Page Cleaner Thread: flush dirty page.

Memory:

1. buffer pool: data page, index page, insert buffer, adaptive hash index, lock info, data dictionary.
2. LRU List, Free List, and Flush List.
3. redo log buffer
4. additional memory pool

Checkpoint