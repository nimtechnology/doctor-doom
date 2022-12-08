# Doctor Doom
![Dr.Doom](./images/drdoom-removebg-preview.png)

## Description
Doctor Doom will destroy files or folder which are:
- Live longer than certain time (default 30 days)
- Have a size bigger than certain size (default 100MB)
- Have a certain extension (default .txt)
- Have a certain name (default .txt)
Doctor Doom will alway find file victims in recursive way. It will not destroy the folder itself.
Doctor Doom will only destroy folder if it is empty (of course, match the conditions).

## Environment
- `doom_path`: The root folder path, where Dr.Doom will look for files to destroy
- `circle`: The time interval (in time unit, integer ) between each Dr.Doom run. Cron tab definition ex: `0 0 * * *` (every day at midnight)
- `doom_export`: The path where Dr.Doom will export the list of files it destroyed. Default is `./doom_victims.log`

## Rules (These rule will alway use the OR logic)
- `age`: The time (in time unit) a file must be older than to be destroyed. Default is 30d
  - `d`: day
  - `h`: hour
  - `m`: minute
- `size`: The size (in size unit) a file must be bigger than to be destroyed. Default is 100MB
  - `B`: byte
  - `K`: kilobyte
  - `M`: megabyte
  - `G`: gigabyte
  - `T`: terabyte
- `name`: The name of the file to be destroyed. Default is `.*`. Can use regex
- `extension`: The extension of the file to be destroyed. Default is `.*`. Can use regex

### Rule priority
- `Age` > `Size` > `Name` > `Extension`

## Example 1
```yaml
# Example 1
doom_path: /home/user
circle: 0 0 * * *
doom_export: /home/user/doom_victims.log
rules:
  age: 30d
  size: 100M
  name: "*"

# Meaning: Dr.Doom will destroy all files in /home/user that are:
# - Live longer than 30 days
# - Have a size bigger than 100MB
# - Have a name that matches regex .*
# - Have a extension that matches regex .txt
# Dr.Doom will run every day at midnight and export the list of files it destroyed to /home/user/doom_victims.log
# The destroy process will use the OR logic between the rules
```

## Example 2
```yaml
# Example 2
doom_path: /home/user
circle: * 14 * * *
doom_export: /home/user/doom_victims.log
rules:
  age: 30d
  size: 10M
  name: "/^victim/"

# Meaning: Dr.Doom will destroy all files in /home/user that are:
# - Live longer than 30 days
# - Have a size bigger than 10MB
# - Have a name that matches regex /^victim/
# - Have a extension that matches regex .txt
# Dr.Doom will run every day at 2pm and export the list of files it destroyed to /home/user/doom_victims.log
```

## Override default value

### Using environment variable

#### Docker container
```bash
docker run -d --name dr-doom -e DOOM_PATH="/home_user" -e CIRCLE="0 0 * * *" \
-e DOOM_EXPORT="/home_user/doom_victims.log" -e RULES_AGE="30d" -e RULES_SIZE="100M" \
-e RULES_NAME=".*" -v /home/user:/home_user \
--restart unless-stopped doctor-doom:latest doom conquer
```

#### Docker compose
```yaml
version: "3.7"
services:
  dr-doom:
    image: doctor-doom:latest
    container_name: dr-doom
    environment:
      - DOOM_PATH="/home_user"
      - CIRCLE="0 0 * * *"
      - DOOM_EXPORT="/var/log/doctor-doom/doom_victims.log"
      - RULES_AGE="30d"
      - RULES_SIZE="100M"
      - RULES_NAME="*"
    volumes:
      - /home/user:/home_user
      - /var/log:/var/log
    restart: unless-stopped
```