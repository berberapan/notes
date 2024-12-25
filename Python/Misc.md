##### Create virtual environment at top level of project

`python3 -m venv venv`

To activate the virtual environment

`source venv/bin/activate`

##### Dependencies

You can have dependencies in a txt-file called `requirements.txt`
To install the requirements you then just give this command
`pip install -r requirements.txt`

##### Regex

Regex is a way to search for patterns in text. To use regex in python you import the **re** module.

Findall method returns all matches in a list. The **r** you use like an f in a format string, it tells Python to treat the string as a raw string.

**To find a specific word**
```python
import re

text = "Heja DIF - gör mål!"
matches = re.findall(r"DIF", text)
print(matches) #['DIF']
```

`\d` matches any digit. 
`{2}` means exact number of preceding character, two in this case.

**To find a Swedish PN**
```python
text = "My pn is 670506-0954 and yours 810102-2382"
matches = re.findall(r"\d{6}-\d{4}", text)
print(matches) #['670506-0095', '810102-2382']
```

`\( \)` escaped parentheses you want to match.
`(.*?)` matches any number of characters. 

**Text between parentheses**
```python
text = "I have a (cat) and a (dog)"
matches = re.findall(r"\((.*?)\)", text)
print(matches) # ['cat', 'dog']
```

`\w` matches any word character.
`+` means one or more of the preceding character.
`@` literal for an 'at'
`\.` is a literal for `.` that we want to match. `.` is a special character in regex.

**Simple regex for email**
```python
text = "My email is joe@example.com and my friend's email is schmoe@example.com"
matches = re.findall(r"\w+@\w+\.\w+", text)
print(matches) # ['joe@example.com', 'schmoe@example.com']
```
