# 11.7 Практическая работа

## Задание 1: Работа с жёсткими и символическими ссылками

```bash
sudo vim delete.sh

#!/bin/bash

if [ $# -eq 0 ]; then
    echo "Usage: $0 <file>"
    exit 1
fi

file=$1

if [ -L "$file" ]; then
    original_file=$(readlink "$file") # Get original file
    rm "$file" # Remove symbolic link
    echo "Only the link has been removed: $file"
    echo "Original file: $original_file"
    
elif [ -f "$file" ]; then
    inodes=$(ls -i "$file" | awk '{print $1}') # Get file inode
    links=$(find . -inum "$inodes") # Find all hard links
    rm "$file" # Remove file
    echo "File removed: $file"
    echo "List of hard links to this file:"
    echo "$links"
else
    echo "File does not exist or is not supported."
    exit 1
fi


sudo chmod +x delete.sh

echo "This is a test file." > testfile.txt
ln -s testfile.txt symlink_to_testfile
ln testfile.txt hardlink_to_testfile
ls -l testfile.txt symlink_to_testfile hardlink_to_testfile

./delete.sh symlink_to_testfile
./delete.sh hardlink_to_testfile
./delete.sh testfile.txt
./delete.sh non_existing_file.txt


echo "This is a test file." > test/testfile.txt
ln -s test/testfile.txt test/symlink_to_testfile
ln test/testfile.txt test/hardlink_to_testfile
ls -l test/testfile.txt test/symlink_to_testfile test/hardlink_to_testfile

./delete.sh test/symlink_to_testfile
./delete.sh test/hardlink_to_testfile
./delete.sh test/testfile.txt
./delete.sh test/non_existing_file.txt
```
### Результаты работы

![](DevOps/WORKS/files/1_1.png)

![](DevOps/WORKS/files/1_2.png)

![](DevOps/WORKS/files/1_3.png)

## Задание 2: Создание systemd-юнита

```bash
sudo vim cleanup.sh

#!/bin/bash

find ~/TRASH -type f -mtime +1 -exec rm {} \;

chmod +x ~/cleanup.sh


sudo vim /etc/systemd/system/cleanup.service

[Unit]
Description=Cleanup old files in TRASH directory

[Service]
Type=oneshot
ExecStart=/home/your_username/cleanup.sh

[Install]
WantedBy=multi-user.target


sudo vim /etc/systemd/system/cleanup.timer

[Unit]
Description=Run cleanup service every hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target


sudo systemctl enable cleanup.timer
sudo systemctl start cleanup.timer
systemctl status cleanup.timer
sudo systemctl list-timers --all
```

### Результаты работы


![](DevOps/WORKS/files/2_cleanup.service.png)

![](DevOps/WORKS/files/2_cleanup.sh.png)

![](DevOps/WORKS/files/2_cleanup.timer.png)

![](DevOps/WORKS/files/2_status.png)

## Задание 3: Преобразование символических ссылок

```bash
sudo vim convert_symlink.sh

#!/bin/bash

if [ $# -eq 0 ]; then
    echo "Usage: $0 <symbolic_link>"
    exit 1
fi

symlink=$1

if [ -L "$symlink" ]; then
    original_file=$(readlink "$symlink")
    
    if [ -f "$original_file" ]; then
        cp "$original_file" "$symlink"
        echo "The symbolic link '$symlink' has been converted to a regular file with the contents of the original '$original_file'."
    else
        echo "The original file '$original_file' does not exist."
    fi
else
    echo "'$symlink' is not a symbolic link."
    exit 1
fi

sudo chmod +x convert_symlink.sh

echo "This is a test file." > testfile.txt
ln -s testfile.txt symlink_to_testfile

./convert_symlink.sh symlink_to_testfile
cat symlink_to_testfile
```

### Результаты работы

![](DevOps/WORKS/files/3_1.png)

![](DevOps/WORKS/files/3_2.png)

## Задание 4: Работа с символическими ссылками

#### Что произойдёт, если скопировать символическую ссылку из одной папки в другую запуском команды вида `cp symlink /path/to/new/dir`?

Будет создана новая символическая ссылка, указывающая на тот же оригинальный файл, но в новом каталоге.
#### Как скопировать символическую ссылку правильно?

Для правильного копирования символической ссылки нужно использовать команду:
```bash
cp -P symlink /path/to/new/dir
```
Это создаст копию символической ссылки, а не содержимого, на которое она ссылается.
