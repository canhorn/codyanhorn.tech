This is the personal blog of Cody Merritt Anhorn. (hi)

# Development  

Run jekyll through server to test deployed pages
~~~
docker run --rm --volume="$PWD\:/srv/jekyll" -p 4000:4000 -it jekyll/minimal:pages jekyll serve --watch --incremental --force_polling
~~~
