Cloneing instruction:

# Step 1: Clone the repo
```bash
git clone https://github.com/nhatvo1502/MyPaperModSite.git
```

# Step 2: Clone the Papermod submodel
```bash
git submodule update --init --recursive
```

# Step 3: Install hugo
```bash
sudo snap install hugo
```

# Step 4: run hugo local
```bash
hugo server -D
```

# Step 5: publish
```bash
git add .
git commit -m 'commit'
git push
```
