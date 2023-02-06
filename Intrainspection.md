# Summary 
For this challenge we are tasked with getting a flag from a password protected zip file(multiple of them actually). In this case the code has this for deciding the passwords of the zip files
```python 
passwords = itertools.product(string.ascii_letters, repeat=4)
```
This means that the passwords are just 4 digit codes with just ASCII letters, meaning it is feasable to brute force it.

# Entry 
Here is a script that does just that for the given zip files(mostly feasable sine the name of the next file is just the password of the previous).
```python
import re
import subprocess
import time

intra_res = subprocess.run(["unzip", "intrainspection.zip"], capture_output=True)

zipname = "Safe.zip"
res = ""

password_regex = re.compile(r"pw == (\w{4})")
inner_regex = re.compile(r"extracting: (\w{4}\.zip)")

zip_start = time.perf_counter()

while "docx" not in res:
    bruteforce_res = subprocess.run(["fcrackzip", "-c", "aA", "-p", "aaaa", "-u", zipname], capture_output=True, text=True)
    zip_password = password_regex.search(bruteforce_res.stdout).group(1)
    print(f"Password for {zipname}: {zip_password}")
    res = subprocess.run(["unzip", "-P", zip_password, zipname], capture_output=True, text=True).stdout
    zip_match = inner_regex.search(res)

    if zip_match is not None:
        zipname = zip_match.group(1)
    
total_time = round(time.perf_counter() - zip_start)
minutes, seconds = divmod(total_time, 60)

print(f"finished decrypting zips! took {minutes} minutes and {seconds} seconds")

```
This uses the fcrackzip program which is used to bruteforce zip file passwords. It still takes a bit since it has 30 layers of zip files, around 14 minutes on my computer. But at the end of that we have a document. However if we remeber that even docx files can be considered zip files, we can peek further and find a xml file named behind the curtain, which is just jumbled data until we also treat it as a zip file which allows us to find a qr code. Reading this qr code just gives a bunch of data, but it has a ELF application header, and so when we run it, we get the flag printed out. Finishing this challenge.