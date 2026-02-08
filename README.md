# Clone
```bash
git clone https://github.com/nhatvo1502/MyPaperModSite.git
```

# Install Hugo + Papermod submodel
```bash
git submodule update --init --recursive
sudo snap install hugo
```

# Run hugo local (-D for draft)
```bash
hugo server -D
```

# Posting
Simply pushing code will trigger GHA. GHA is already bundled with hosting credentials and will take care of posting automatically.
