---
layout: post
title: Use Git Hooks to Automate Necessary but Annoying Tasks
modified: 2014-09-17
---

Certain tasks like updating dependencies or migrating a database must be done
after pulling code or checking out a branch. Other tasks such as re-indexing our
`ctags` improve our development experience. Both kinds of tasks are easy to
forget to do and are therefore error-prone. To address the problem, we've
recently added a standard, extensible set of [`git`][2] hooks to our
[dotfiles][1] in order to automate necessary, but annoying tasks.

## Git Hooks

Git has a commonly under-utilized feature: [hooks][hooks]. You can think of a
hook as an event that gets triggered before and after various stages of revision
control process. Some hooks of note are:

- `prepare-commit-msg` - Fires before the commit message prompt.
- `pre-commit` - Fires before a `git commit`.
- `post-commit` - Fires after a `git commit`.
- `post-checkout` - Fires after changing branches.
- `post-merge` - Fires after merging branches.
- `pre-push` - Fires before code is pushed to a remote.

## Extending our Hooks

Our dotfiles' [convention for extension][3] is to place our custom hooks in
`{pre,post}-$EVENT` files within our `~/.git_template.local/hooks` directory.
Now, anything we add to those hook files will be automatically executed, running
tasks that we normally would forget.

## What tasks do you commonly forget

> I forget to re-index my `ctags`!

Lucky for you, we've set up `git` to [re-index your `ctags`][4] after each `git`
command.

> I always forget to run `bundle install` after switching branches!

Automatically install new gems:

```sh
# ~/.git_template.local/hooks/post-checkout

# use `hookup` gem if it's installed
if command -v hookup > /dev/null; then
  hookup post-checkout "$@"
else
  # otherwise, do it yourself
  [ -f Gemfile ] && bundle install > /dev/null &
fi
```

> I never remember to run pending migrations!

Automatically run your migrations:

```sh
# ~/.git_template.local/hooks/post-checkout

# use `hookup` gem if it's installed
if command -v hookup > /dev/null; then
  hookup post-checkout "$@"
else
  # otherwise, do it yourself
  [ -f db/schema.rb ] && bin/rake db:migrate > /dev/null &
fi
```

> I document my API with [fdoc](https://github.com/square/fdoc), but I forget to
> generate the pages!

Automatically generate the <abbr title="HyperText Markup Language">HTML</abbr>
docs:

```sh
# ~/.git_template.local/hooks/post-checkout

bin/fdoc convert ./spec/fixtures --output=./html > /dev/null &
```

> I really like Go's commitment to a standard code format, but I constantly
> forget to format my files!

Run `go fmt` before you commit:

```sh
# ~/.git_template.local/hooks/pre-commit

gofiles=$(git diff --cached --name-only --diff-filter=ACM | grep '.go$')
[ -z "$gofiles" ] && exit 0

function checkfmt() {
  unformatted=$(gofmt -l $gofiles)
  [ -z "$unformatted" ] && return 0

  echo >&2 "Go files must be formatted with gofmt. Please run:"
  for fn in $unformatted; do
    echo >&2 "  gofmt -w $PWD/$fn"
  done

  return 1
}

checkfmt || fail=yes

[ -z "$fail" ] || exit 1

exit 0
```

> I want my extensive network of friends to know when I'm merging code!

Send out a [Yo][5] every time you merge a branch:

```sh
# ~/.git_template.local/hooks/post-merge

curl --data "api_token=$YO_API_TOKEN" https://api.justyo.co/yoall/ > /dev/null &
```

When we aggressively simplify and automate the tedious parts of the development
process, we can focus on what's important: getting things done.

## What's next

If you found this useful, you might also enjoy:

- [Hookup][6] gem for Rails-specific git hooks, including super-charged versions
  of the Bundler and migrations hooks.
- [Silver Searcher Tab Completion with Exuberant Ctags][7]

[hooks]: http://git-scm.com/book/en/Customizing-Git-Git-Hooks
[1]: https://github.com/thoughtbot/dotfiles/commit/cbdcbce01dea1ab3850be2311f33f00d75f6088b
[2]: http://git-scm.com/
[3]: https://github.com/thoughtbot/dotfiles/blob/85e9f22e48718b246235fd0b409be0e4d21eddbc/README.md#make-your-own-customizations
[4]: https://github.com/thoughtbot/dotfiles/commit/cbdcbce01dea1ab3850be2311f33f00d75f6088b
[5]: http://www.justyo.co/
[6]: https://github.com/tpope/hookup
[7]: http://robots.thoughtbot.com/silver-searcher-tab-completion-with-exuberant-ctags
