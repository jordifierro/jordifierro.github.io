http://jordifierro.com/

# Setup

To test it locally:
```
docker run -t --volume="$PWD:/srv/jekyll" -p 4000:4000 -t jekyll/jekyll:3.8 bash -c "jekyll serve"
```

To build production statics:
```
docker run -t --volume="$PWD:/srv/jekyll" --env JEKYLL_ENV=production jekyll/jekyll:3.8 bash -c "jekyll build --trace"
```

_Trick_: add `--volume="$PWD/../bundle:/usr/local/bundle"` to reduce installation time.

