---
layout: post
title: python regular
date: 2020-11-02
tags: python regular
---
每次要用時才查還搞不懂 .... 下面還有習題

```
中介字元	說明
.	除了新行符號外的任何字元，例如 '.' 配對除了 '\n' 之外的任何字元。
^	字串開頭的子字串或排除指定字元或群組，例如 'a[^b]c' 配對除了 'abc' 之外的任何 a 開頭 c 結尾的三字元組合。
$	字串結尾的子字串，例如 'abc$' 配對以 'abc' 結尾的字串。
*	單一字元或群組出現任意次數，例如 'ab*' 配對 'a' 、 'ab' 或 'abb' 等等。
+	單一字元或群組出現至少一次，例如 'ab+' 配對 'ab' 或 'abb' 等等。
?	單一字元或群組 0 或 1 次，例如 'ab+' 配對 'a' 或 'ab' 。
{m,n}	單一字元或群組的 m 到 n 倍數，例如 'a{6}' 為連續六個 'a' ， 'a{3,6}' 為三到六個 'a' 。
[]	對中括弧內的字元形成集合，例如 '[a-z]' 為所有英文小寫字母。
\	特別序列的起始字元。
|	單一字元或群組的或，例如 'a|b' 為 'a' 或 'b' 。
()	對小括弧內的字元形成群組。
```

```
\number	群組的序數
\A	字串的開頭字元。
\b	作為單字的界線字元，例如 r'\bfoo\b' 配對 'foo' 或 'bar foo baz' 。
\B	作為字元的界線字元，例如 r'py\B' 配對 'python' 或 'py3' 。
\d	數字，從 0 到 9 。
\D	非數字。
\s	各種空白符號，包括新行符號 \n 。
\S	非空白符號。
\w	任意文字字元，包括數字。
\W	非文字字元，包括空白符號。
\Z	字串的結尾字元。
```

```
compile(pattern)	以配對形式字串 pattern 當參數，回傳 re.compile() 物件。
search(pattern, string, flags=0)	從 string 中找尋第一個配對形式字串 pattern ，找到回傳配對物件，沒有找到回傳 None 。
match(pattern, string, flags=0)	判斷配對形式字串 pattern 是否與 string 的開頭相符，如果相符就回傳配對物件，不相符就回傳 None 。
fullmatch(pattern, string, flags=0)	判斷 string 是否與配對形式字串 pattern 完全相符，如果完全相符就回傳配對物件，不完全相符就回傳 None 。
split(pattern, string, maxsplit=0, flags=0)	將 string 以配對形式字串 pattern 拆解，結果回傳拆解後的串列。
findall(pattern, string, flags=0)	從 string 中找到所有的 pattern ，結果回傳所有 pattern 的串列。
finditer(pattern, string, flags=0)	從 string 中找到所有的 pattern ，結果回傳所有 pattern 的迭代器。
sub(pattern, repl, string, count=0, flags=0)	依據 pattern 及 repl 對 string 進行處理，結果回傳處理過的新字串。
subn(pattern, repl, string, count=0, flags=0)	依據 pattern 及 repl 對 string 進行處理，結果回傳處理過的序對。
escape(pattern)	將 pattern 中的特殊字元加入反斜線，結果回傳新字串。
purge()	清除正規運算式的內部緩存。
```

題目 :
```
The phone number field contains U.S. phone numbers, and needs to be modified to the international format, 
with "+1-" in front of the phone number. Fill in the regular expression, using groups, 
to use the transform_record function to do that.
```

```
import re
def transform_record(record):
  new_record = re.sub(r",(\d+)",r",+1-\1",record)  
  #把 ','+數字的改成 ... ',+1-' 加數字.... 那個 \1 就是原來的括弧資訊
  #用括弧可以將資訊帶到後面 \1 , \2 這樣用   
  return new_record

print(transform_record("Sabrina Green,802-867-5309,System Administrator")) 
# Sabrina Green,+1-802-867-5309,System Administrator

print(transform_record("Eli Jones,684-3481127,IT specialist")) 
# Eli Jones,+1-684-3481127,IT specialist

print(transform_record("Melody Daniels,846-687-7436,Programmer")) 
# Melody Daniels,+1-846-687-7436,Programmer

print(transform_record("Charlie Rivera,698-746-3357,Web Developer")) 
# Charlie Rivera,+1-698-746-3357,Web Developer
```
```
The multi_vowel_words function returns all words with 3 or more consecutive vowels
 (a, e, i, o, u). Fill in the regular expression to do that.
```
```
import re
def multi_vowel_words(text):
  pattern = r'(\w*[aeiou]{3,}\w*)'
  #匹配 中間為 aeiou 的字且 要三個字以上
  result = re.findall(pattern, text)
  return result

print(multi_vowel_words("Life is beautiful")) 
# ['beautiful']

print(multi_vowel_words("Obviously, the queen is courageous and gracious.")) 
# ['Obviously', 'queen', 'courageous', 'gracious']

print(multi_vowel_words("The rambunctious children had to sit quietly and await their delicious dinner.")) 
# ['rambunctious', 'quietly', 'delicious']

print(multi_vowel_words("The order of a data queue is First In First Out (FIFO)")) 
# ['queue']

print(multi_vowel_words("Hello world!")) 
# []
```

```
import re
def transform_comments(line_of_code):
  result = re.sub(r'##*',r'//', line_of_code)
  return result

print(transform_comments("### Start of program")) 
# Should be "// Start of program"
print(transform_comments("  number = 0   ## Initialize the variable")) 
# Should be "  number = 0   // Initialize the variable"
print(transform_comments("  number += 1   # Increment the variable")) 
# Should be "  number += 1   // Increment the variable"
print(transform_comments("  return(number)")) 
# Should be "  return(number)"
```

```
import re
def convert_phone_number(phone):
  result = re.sub(r"(\w*) (\d+)-(\d+)", r"\1 (\2) \3", phone)
  return result

print(convert_phone_number("My number is 212-345-9999.")) # My number is (212) 345-9999.
print(convert_phone_number("Please call 888-555-1234")) # Please call (888) 555-1234
print(convert_phone_number("123-123-12345")) # 123-123-12345
print(convert_phone_number("Phone number of Buckingham Palace is +44 303 123 7300")) # Phone number of Buckingham Palace is +44 303 123 7300
```

```
import re
def show_time_of_pid(line):
  pattern = r'(\w+ \d+ \d+:\d+:\d+) \w+.\w+ \w*[=]{0,}\w+\[(\d+)'
  result = re.search(pattern, line)
  #return result[1],result[2],
  return '{} pid:{}'.format(result[1],result[2])

print(show_time_of_pid("Jul 6 14:01:23 computer.name CRON[29440]: USER (good_user)")) # Jul 6 14:01:23 pid:29440
print(show_time_of_pid("Jul 6 14:02:08 computer.name jam_tag=psim[29187]: (UUID:006)")) # Jul 6 14:02:08 pid:29187
print(show_time_of_pid("Jul 6 14:02:09 computer.name jam_tag=psim[29187]: (UUID:007)")) # Jul 6 14:02:09 pid:29187
print(show_time_of_pid("Jul 6 14:03:01 computer.name CRON[29440]: USER (naughty_user)")) # Jul 6 14:03:01 pid:29440
print(show_time_of_pid("Jul 6 14:03:40 computer.name cacheclient[29807]: start syncing from \"0xDEADBEEF\"")) # Jul 6 14:03:40 pid:29807
print(show_time_of_pid("Jul 6 14:04:01 computer.name CRON[29440]: USER (naughty_user)")) # Jul 6 14:04:01 pid:29440
print(show_time_of_pid("Jul 6 14:05:01 computer.name CRON[29440]: USER (naughty_user)")) # Jul 6 14:05:01 pid:29440
```

```
#!/usr/bin/env python3

#Import libraries
import csv
import re

def contains_domain(address, domain):
  """Returns True if the email address contains the given domain,
    in the domain position, false if not."""
  domain_pattern = r'[\w\.-]+@'+domain+'$'
  if re.match(domain_pattern, address):
    return True
  return False

def replace_domain(address, old_domain, new_domain):
  """Replaces the old domain with the new domain in
    the received address."""
  old_domain_pattern = r'' + old_domain + '$'
  address = re.sub(old_domain_pattern, new_domain, address)
  return address

def main():
  """Processes the list of emails, replacing any instances of the old domain with the new domain."""
  old_domain, new_domain = 'abc.edu', 'xyz.edu'
  csv_file_location = '/home/student-04-810d01400547/data/user_emails.csv'
  report_file = '/home/student-04-810d01400547/data' + '/updated_user_emails.csv'
  user_email_list = []
  old_domain_email_list = []
  new_domain_email_list = []

  with open(csv_file_location, 'r') as f:
    user_data_list = list(csv.reader(f))
    user_email_list = [data[1].strip() for data in user_data_list[1:]]

    for email_address in user_email_list:
      if contains_domain(email_address, old_domain):
        old_domain_email_list.append(email_address)
        replaced_email = replace_domain(email_address,old_domain,new_domain)
        new_domain_email_list.append(replaced_email)

    email_key = ' ' + 'Email Address'
    email_index = user_data_list[0].index(email_key)

    for user in user_data_list[1:]:
      for old_domain, new_domain in zip(old_domain_email_list, new_domain_email_list):
        if user[email_index] == ' ' + old_domain:
          user[email_index] = ' ' + new_domain
  f.close()

  with open(report_file, 'w+') as output_file:
    writer = csv.writer(output_file)
    writer.writerows(user_data_list)
    output_file.close()
main()
```
