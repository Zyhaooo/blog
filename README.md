## local server

```sh
docker run -d --name=blog-server -p 4000:4000 -v blog:/srv/jekyll jekyll/jekyll jekyll server --watch
```