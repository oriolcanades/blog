# blog
My personal blog

# Run the project locally

To run the project locally, you need to have Ruby and Jekyll installed on your computer. If you don't have them installed, follow the instructions in the [official Jekyll documentation](https://jekyllrb.com/docs/installation/).

If you have Ruby and Jekyll installed, follow these steps:

```bash
bundle exec jekyll serve
```

# Stop the project locally

To stop the project, press `Ctrl + C` in the terminal. If this does not work, close the terminal window.

```bash
ps aux |grep jekyll |awk '{print $2}' | xargs kill -9
```


