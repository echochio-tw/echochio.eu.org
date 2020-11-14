---
layout: post
title: python unit test
date: 2020-11-15
tags: python unittest
---
unit test 沒用過既然老師教就學一下

大概就是輸入然後測試結果 .. 修改到沒錯為止

user_emails.csv
```
Blossom Gill,blossom@abc.edu,
Hayes Delgado,nonummy@abc.edu
Petra Jones,ac@abc.edu
Oleg Noel,noel@abc.edu
Ahmed Miller,ahmed.miller@abc.edu
Macaulay Douglas,mdouglas@abc.edu
Aurora Grant,enim.non@abc.edu
Madison Mcintosh,mcintosh@abc.edu
Montana Powell,montanap@abc.edu
Rogan Robinson,rr.robinson@abc.edu
Simon Rivera,sri@abc.edu
Benedict Pacheco,bpacheco@abc.edu
Maisie Hendrix,mai.hendrix@abc.edu
Xaviera Gould,xlg@abc.edu
Oren Rollins,oren@abc.edu
Flavia Santiago,flavia@abc.edu
Jackson Owens,jacksonowens@abc.edu
Britanni Humphrey,britanni@abc.edu
Kirk Nixon,kirknixon@abc.edu
Bree Campbell,breee@abc.edu
```

emails.py
```
#!/usr/bin/env python3

import sys
import csv


def populate_dictionary(filename):
  """Populate a dictionary with name/email pairs for easy lookup."""
  email_dict = {}
  with open(filename) as csvfile:
    lines = csv.reader(csvfile, delimiter = ',')
    for row in lines:
      name = str(row[0].lower())
      email_dict[name] = row[1]
  return email_dict

def find_email(argv):
  """ Return an email address based on the username given."""
  # Create the username based on the command line input.
  try:
    fullname = str(argv[1] + " " + argv[2])
    # Preprocess the data
    email_dict = populate_dictionary('/home/student-01-302faad05fe8/data/user_emails.csv')
    # If email exists, print it
    if email_dict.get(fullname.lower()):
      return email_dict.get(fullname.lower())
    else:
      return "No email address found"
    # Find and print the email
  except IndexError:
    return "Missing parameters"

def main():
  print(find_email(sys.argv))

if __name__ == "__main__":
  main()
```

emails_test.py
```
#!/usr/bin/env python3
import unittest
from emails import find_email
class EmailsTest(unittest.TestCase):
  def test_basic(self):
    testcase = [None, "Bree", "Campbell"]
    expected = "breee@abc.edu"
    self.assertEqual(find_email(testcase), expected)
  def test_one_name(self):
    testcase = [None, "John"]
    expected = "Missing parameters"
    self.assertEqual(find_email(testcase), expected)
  def test_two_name(self):
    testcase = [None, "Roy","Cooper"]
    expected = "No email address found"
    self.assertEqual(find_email(testcase), expected)

if __name__ == '__main__':
  unittest.main()
```
