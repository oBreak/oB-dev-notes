# Dev Notes

## Setup

#### Source control: 

|Program                |                       Notes|
|-----------------------|----------------------------|
|Sourcetree             |I like the Mac version of this. Windows version seems fairly comparable, but I don't have as much experience with it.    |
|git                    |Command line     |

#### IDEs:

|Program                |                       Notes|
|-----------------------|----------------------------|
|Visual Studio Code     |Automatic installation of powershell, python, etc. plugins  |
|PyCharm                |Nice IDE for python, environment setup is complicated     |



#### PyCharm Environment

- venv created in `directory`

#### Connecting (SourceTree on Windows)

Installing Sourcetree on Windows is an adventure compared to setting it up on a Mac.

#### Versioning (Sourcetree on Mac)

Ran...

```bash
defaults write com.apple.finder AppleShowAllFiles TRUE
killall Finder
```

...to show .gitignore files in Finder.

#### PIP

Upgrading pip:

- Upgraded pip to 10.0.1 -- via `python -m pip install --upgrade pip`
- Upgraded pip to 19.2.2 -- via `pip install --upgrade pip`
- Upgraded pip to 20.1 -- via `pip install --upgrade pip`

Installing packages:

- Downloading a package `pip install <name of module>`

### PowerShell



#### Powershell on Mac

https://wilsonmar.github.io/powershell-on-mac/ - Helpful

- Ran `xcode-select --install` and had to install command line tools
- Ran `brew cask install powershell` (Took a while to update homebrew)
- Check installed version with `$psversiontable`
- For AD use cases on Mac: I don't know yet.

#### Powershell (on Windows)

- For AD use cases on Windows Server 2016: Add Windows Features RSAT > RAT > AD DS and AD LDS Tools > Active Directory 
module for Windows Powershell (check this)


#### Visual Studio Code for Mac

https://code.visualstudio.com/docs?dv=osx

- Recommended download from Microsoft
- Installed powershell extension (recommended by app)



# Useful 

### Script Hashbang

- `#!/usr/lib/env python`
- Flesh this out^

### Mac OSX

- `nc ipaddress port` - Netcat is replacing telnet on High Sierra (OX 10.13), this 
is a way to check if a port is open.
- `sudo spctl --master-disable` - Disables gatekeeper (unsigned applications can run)
- `sudo spctl --master-enable` - Enables gatekeeper (unsigned applications can't run)
- `sudo -i` - Removes requirement to type password for each sudo command 
in terminal window until exit or terminal window is closed.
- 

### Web and Native Tools

- Downloaded Postman for API testing.

### OpenSSL

- https://www.sslshopper.com/article-most-common-openssl-commands.html - Common OpenSSL commands;
includes things like conversion of certificate types, debugging, and creating new certificate
signing requests. Helpful resource.

### Python3 packages (python3)

Default

- os
- sys
- time
- requests
- subprocess
- socket

Not default 

- certifi
- pathlib
- configparser
- keyring
- pycrypto
- pysqlite3
- setuptools
- urllib3



### Powershell

|Program                |                       Notes|
|-----------------------|----------------------------|
|Get-ADUser                         |https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-aduser?view=winserver2012-ps  |
|Search-ADAccount –LockedOut        |Shows all accounts locked out in domain     |

To view the paths that are specified in the PSModulePath variable, type the following command:

$env:PSModulePath









## Troubleshooting
For notes around troubleshooting Python. May contain code but will mostly be documentation. 
I have both Python 2.7 and 3.6 installed so it will be applicable to both, in theory. I will
try to be specific, where possible.

#### Git commits after Mojave upgrade

https://stackoverflow.com/questions/52522565/git-is-not-working-after-macos-mojave-update-xcrun-error-invalid-active-devel


#### SSL Errors


When installing python 3.6, an error was received.

Then, running a package install for python 3:

```bash
Computer:Project Username$ pip3 install package -v
Created temporary directory: /private/var/folders/b_/f0l36myn0zxg0pw7khm17xh40000gp/T/pip-ephem-wheel-cache-7p_b0q3_
Created temporary directory: /private/var/folders/b_/f0l36myn0zxg0pw7khm17xh40000gp/T/pip-install-nfrw2gsg
Collecting PyCrypto
  1 location(s) to search for versions of package:
  * https://pypi.org/simple/pycrypto/
  Getting page https://pypi.org/simple/package/
  Looking up "https://pypi.org/simple/package/" in the cache
  No cache entry available
  Starting new HTTPS connection (1): pypi.org
  Incremented Retry for (url='/simple/package/'): Retry(total=4, connect=None, read=None, redirect=None, status=None)
  Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:833)'),)': /simple/package/
  Starting new HTTPS connection (2): pypi.org
  Incremented Retry for (url='/simple/package/'): Retry(total=3, connect=None, read=None, redirect=None, status=None)
  Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:833)'),)': /simple/package/
```

I stole some of the code out of the 'Install Certificates.command' included with the Python distribution and ran...

```python
#!/bin/sh

import certifi
import os
import ssl
import stat
STAT_0o775 = ( stat.S_IRUSR | stat.S_IWUSR | stat.S_IXUSR
             | stat.S_IRGRP | stat.S_IWGRP | stat.S_IXGRP
             | stat.S_IROTH |                stat.S_IXOTH )
def main():
    openssl_dir, openssl_cafile = os.path.split(
        ssl.get_default_verify_paths().openssl_cafile)
    os.chdir(openssl_dir)
    #print(openssl_dir)  -- yields /Library/Frameworks/Python.framework/Versions/3.6/etc/openssl
    relpath_to_certifi_ca_file = os.path.relpath(certifi.where())
    try:
        os.remove(openssl_cafile)
    except FileNotFoundError:
        print("error")
        pass
    os.symlink(relpath_to_certifi_ca_file, openssl_cafile)
    f = open(openssl_cafile)
    f.readline()
    for line in f:
        print (line)
    os.chmod(openssl_cafile, STAT_0o775)
    print ("--update complete")
    return

main()
```

Now I at least know where to look for the cafile. 

```bash
openssl s_client -connect pypi.org:443
```

Shows that I'm going through a proxy. I attempted to load in the proxy certificate unsuccessfully.

Eventually, I found that if you add:

```bash
[global]

trusted-host = pypi.python.org
               pypi.org
               files.pythonhosted.org
```

To /Library/Application Support/pip/pip.conf (Had to create the file and directory), it succeeds.



