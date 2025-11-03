---

description: Run a command multiple times with different values

---

For each value in the list: $1
Run the command: /$2 with each value and with an extra arg $3

Example: If given "value1, value2, value3" and "mycommand" and extra arg is "describing-the-ui"
Execute:

- /mycommand value1 describing-the-ui
- /mycommand value2 describing-the-ui
- /mycommand value3 describing-the-ui

ARGUMENTS: $1 (comma-separated values), $2 (command name), $3 (extra arg)
