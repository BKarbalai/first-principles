# Command Line Cheatsheet

Source: Codédex Command Line cheatsheet. Notes for Linux Command Line (May 23, 2026).

## Change Directory

```sh
cd Desktop
cd projects
```

## List Directory Contents

```sh
ls
```

## Print Working Directory

```sh
pwd
```

## Navigation

```sh
cd Desktop/projects/weather  # Go forward

cd ..     # Go back one directory
cd ../..  # Go back two directories
```

## Print Message

```sh
echo Hello
```

## Display File Contents

```sh
cat welcome.txt
```

## Clear Terminal Screen

```sh
clear
```

## Keyboard Shortcuts

- Command history: `↑` `↓`
- Autocomplete: `tab`

## Make New Directory

```sh
mkdir homework
```

## Create New File

```sh
touch grocery-list.txt
```

## Overwrite Content

```sh
echo "no experience" > resume.txt
```

## Add to Content

```sh
echo "this is my resume" >> resume.txt
```

## Overwrite File with Another File

```sh
cat resume-2025.txt > resume-2026.txt
```

## Append File Content to Another File

```sh
cat goals-25.txt >> goals-26.txt
```

## Move or Rename File/Directory

```sh
mv old-name new-name
```

## Remove File/Directory

```sh
rm file.txt
rm -r directory
```

## Copy File

```sh
cp file-a file-b
```

## My additions

- **`>` destroys, `>>` appends.** `> resume.txt` truncates first, then writes. The classic disaster is `cat a.txt > a.txt` (empties the file before reading it). When in doubt, back up before redirecting.
- **`rm` has no trash can.** Deleted files don't go to recycle bin. `rm -r` on the wrong directory is unrecoverable, so `pwd` + `ls` before any recursive delete, and never `rm -rf /` or `rm -rf ~`.
- **Tab + history are the speed tools.** Tab completes paths (double-tab lists options), `↑`/`↓` recalls commands, `Ctrl+R` searches history. Faster than retyping and fewer typos in destructive commands.
- **Relative vs absolute paths.** `cd projects` works from wherever you are (fragile). `cd ~/projects` or `cd /home/user/projects` works from anywhere (robust). Scripts and notes should use absolute or `~` paths.
- **`ls -a` shows the hidden files.** Dotfiles (`.gitignore`, `.bashrc`) are invisible to plain `ls`. `ls -la` adds permissions and sizes, which is how you spot a non-executable script or a wrong owner.
- **Read the manual first.** `man <command>` (or `<command> --help`) documents every flag on the machine, no internet needed. Faster than guessing flags for `cp`, `mv`, `rm`.
- **Chain deliberately.** `cmd1 && cmd2` runs cmd2 only if cmd1 succeeds (safe for `cd dir && rm file`). `cmd1 ; cmd2` runs both regardless. `|` pipes output into the next command (`ls | grep notes`).

---
[Index](README.md)
