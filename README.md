# Clone:
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
simply pushing code will trigger github action with pre-embedded credentials for posting
