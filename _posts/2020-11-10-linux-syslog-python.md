---
layout: post
title: python linux syslog find word
date: 2020-11-10
tags: linux syslog python regular
---

```
#!/usr/bin/env python3
import sys
import os
import re
def error_search(log_file):
  error = input("What is the error? ")
  returned_errors = []
  with open(log_file, mode='r',encoding='UTF-8') as file:
    for log in  file.readlines():
      error_patterns = ["error"]
      for i in range(len(error.split(' '))):
        error_patterns.append(r"{}".format(error.split(' ')[i].lower()))
      if all(re.search(error_pattern, log.lower()) for error_pattern in error_patterns):
        returned_errors.append(log)
    file.close()
  return returned_errors
def file_output(returned_errors):
  with open(os.path.expanduser('~') + '/data/errors_found.log', 'w') as file:
    for error in returned_errors:
      file.write(error)
    file.close()
if __name__ == "__main__":
  log_file = sys.argv[1]
  returned_errors = error_search(log_file)
  file_output(returned_errors)
  sys.exit(0)
```

```
student-01-f5e9e55878c3@linux-instance:~/scripts$ ./find_error.py ~/data/fishy.log
What is the error? system ERROR
student-01-f5e9e55878c3@linux-instance:~/scripts$ cat ~/data/errors_found.log
July 31 00:33:31 mycomputername system[25588]: ERROR Out of yellow ink, specifically, even though you want grayscale
July 31 08:21:11 mycomputername system[78476]: ERROR Error running Python2.exe: Segmentation Fault (core dumped)
July 31 08:55:07 mycomputername system[24562]: ERROR Process failed
July 31 09:47:30 mycomputername system[61238]: ERROR I am error
July 31 09:54:04 mycomputername system[18585]: ERROR 404 error not found
July 31 10:53:36 mycomputername system[78903]: ERROR Encapsulating packets
July 31 11:19:56 mycomputername system[15816]: ERROR I am error
July 31 12:42:41 mycomputername system[38483]: ERROR AssertionError 'False' is not 'True'
July 31 12:48:04 mycomputername system[68448]: ERROR lp0 on fire
July 31 13:00:34 mycomputername system[62593]: ERROR Out of yellow ink, specifically, even though you want grayscale
July 31 13:14:54 mycomputername system[84324]: ERROR 404 error not found
July 31 13:23:12 mycomputername system[56046]: ERROR Unable to perform package upgrade
July 31 16:36:57 mycomputername system[27219]: ERROR Process failed
July 31 19:44:47 mycomputername system[50340]: ERROR lp0 on fire
July 31 23:17:05 mycomputername system[88316]: ERROR 418: I'm a teapot
July 31 23:20:09 mycomputername system[42918]: ERROR Out of ink
```

```
student-01-f5e9e55878c3@linux-instance:~/scripts$ ./find_error.py ~/data/fishy.log
What is the error? CRON ERROR Failed to start
student-01-f5e9e55878c3@linux-instance:~/scripts$ cat ~/data/errors_found.log
July 31 04:11:32 mycomputername CRON[51253]: ERROR: Failed to start CRON job due to script syntax error. Inform the CRON job owner!
```

```
student-01-f5e9e55878c3@linux-instance:~/scripts$ ./find_error.py ~/data/fishy.log
What is the error? kernel Syntax ERROR
student-01-f5e9e55878c3@linux-instance:~/scripts$ cat ~/data/errors_found.log
July 31 21:48:30 mycomputername kernel[33330]: ERROR Syntax issue
```
