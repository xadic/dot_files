
git:
```sh
echo "# Ricking" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:xadic/Ricking.git
git push -u origin main
```

删除remote *origin*:
```sh
git remote rm origin
```

中文文件夹和文件在linux中乱码问题:
```sh
unzip -O cp936 <your_zip_file>
```

```sh
iconv -f GB2312 -t utf-8 '<origin_file>' -o <converted_file>
```
