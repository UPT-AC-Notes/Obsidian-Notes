# LINUX WALKTHROUGH

stdin -> prg 1 -> stdout / ecran -> (stdin)prg2
echo "ello" > alt_fisier.txt
wc - word count
alt_fisier.txt | wc -m -> word count - characters
rm fisier.txt -> remove
mv fisier.txt alt_fiser.txt -> move

- sudo add user -> adaugare nou utilizator
- su "nou user" -> schimare utilizator
- exit -> iesire din root

- cp dir1/\*.txt dir2 (orice .txt copiat)
- mv dir1/fisier dir1/alt_fisier
- ls -lia -> grupuri de 3 -> user / grup / altcineva
- chmod 700 / 777 -> schimare drepturi de read/write/execute
- chown -change owner
- ps -aux | wc -l -> numar procese

### SSH

.ssh -> autentficare prin chei de ssh

authorised_keys -> fisier cu chei

ssh-keygen.exe -> cheie publica -> .ssh

```zsh
ssh-keygen -b 4096 -t rsa
```

### FIND
```zsh
find . -name *.txt

find . -name "*.txt" | xargs cat > output.txt # to chain arguments
```

### GREP
```zsh
grep -r "text" . # finder
```

### SED
```zsh
cat fisier.txt | sed "s/Petrolieru/Becali/g" > output.txt # search and replace
```

```zsh
de_cate_ori_apare_shrek = "$(cat Shrek.2001.1080p.BluRay.x264-iHD.srt | grep "Shrek" | wc -l)"
```

### RUBY REGEX
```ruby
^[a-z]+(\.[a-z]+)*@[a-z]+\.[a-z]+$
```
