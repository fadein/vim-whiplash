# Whiplash

Easy project switcher for Vim.

## Why?

I switch between software projects frequently. Changing projects can be
a pain in Vim. For example, every time I want to work on a different
project, I have to call `:cd` to change my working directory, flush my
[Command-T](https://github.com/wincent/Command-T) cache, synchronize my
[NERDTree](https://github.com/scrooloose/nerdtree) root directory, and
so on.

Enough with the madness!

I just want a simple way to execute a bunch of commands when I change
projects. Whiplash does that. Want to change projects? It's as easy as:

    :Whiplash project-name

## Installation

Get [Pathogen](https://github.com/tpope/vim-pathogen), and then:

    cd ~/.vim/bundle
    git clone https://github.com/arkwright/vim-whiplash.git

Only tested in Vim 7.4 on OS X using MacVim. It probably doesn't work on
Windows. If you would like Windows support, please open an issue.

## Usage

To change projects, simply invoke the `:Whiplash` command:

    :Whiplash project-name

Where `project-name` is the directory name of the project you want
Whiplash to switch to.

Of course, none of this will work until you complete the configuration
steps below.

## Configuration

Before you can use Whiplash, you need to tell it a little bit about
yourself.

Add this line to your `.vimrc` file:

    let g:WhiplashProjectsDir = "~/projects/"

Replace `~/projects/` with a path to the directory where you keep your
projects.

By default, you can switch projects using the `:Whiplash` command. If
you want to change the name of the command, add this line to your
`.vimrc` file:

    let g:WhiplashCommandName = "Project"

Replace `Project` with the command name that you prefer. With the
configuration shown above, you can invoke Whiplash to switch projects
like so:

    :Project project-name

The `project-name` part is, of course, the name of the project you want
to switch to.

By default, Whiplash doesn't really do anything. That's because you need
to tell it *what* to do when switching projects. Should the current
working directory be reset? Should NERDTree open? You decide! Whiplash
allows you to specify Vimscript configuration files which should be run
when switching projects. Don't worry: it's not as complicated as it
sounds.

By default, Whiplash assumes you want to keep your project configuration
files inside your dotfiles repo, in this directory:

    ~/projects/dotfiles/whiplash-config/

If you want to change the directory where the configuration files are
stored, add this line to your `.vimrc` file:

    let g:WhiplashConfigDir = '/your/whiplash/configuration/dir/'

Replace `/your/whiplash/configuration/dir/` with a path to the directory
where you want to store your Whiplash configuration files.

When switching projects, you provide Whiplash with a project name. That
project name is used to locate a Vimscript configuration file within
your Whiplash configuration directory. For example, if your project name
is `fubar`, and your Whiplash configuration directory is set to
`~/projects/dotfiles/whiplash-config/`, then Whiplash will attempt to
execute all Vim commands inside these three files:

    ~/projects/dotfiles/whiplash-config/pre.vim
    ~/projects/dotfiles/whiplash-config/fubar/config.vim
    ~/projects/dotfiles/whiplash-config/post.vim

Why three files? To give you more control over the project switching
process. The files are executed in the order listed above: `pre.vim`,
then `fubar/config.vim`, then finally `post.vim`.

`pre.vim` and `post.vim` are *global* configuration files. This means
they are executed *every time* you switch projects. The `config.vim`
file is the *project* configuration file: it is executed *only* when
switching *to* the project it is specified for.

Sample `pre.vim`, `post.vim`, and `config.vim` files are provided inside the
[whiplash-config](https://github.com/arkwright/vim-whiplash/tree/master/whiplash-config)
directory to give you inspiration for the cool things that Whiplash can
do.

## Configuration Commands

Whiplash provides special commands to handle common project
configuration tasks. You can use these commands inside your Whiplash
config files.

### :WhiplashCD

The `:WhiplashCD` command changes Vim's current working directory to be
the current Whiplash project directory. It is equivalent to calling `:cd
<your project dir>`, as one would do to change the working directory
back in the pre-Whiplash days, when steam-power was still a disruptive
innovation.

### :WhiplashCopyFile <file_path>

The `:WhiplashCopyFile` command copys the specified file from your
Whiplash project config directory into the main project directory. It
overwrites the destination file if it already exists. An example will
make it easier to understand why this is useful.

Let's say you work with a team on a software project named `fubar`. You
keep a clone of the `fubar` Git repo at `~/projects/fubar`. The team
shares a `.gitignore` file inside the repo. You don't like the way the
shared `.gitignore` file is set up, and you want to customize it. Your
problem is that you aren't allowed to commit your customizations,
because then everyone else on the team will be forced to use them. This
is frustrating for you, because each time you reset your local copy of
the `fubar` repo, you lose all of your `.gitignore` customizations.

`:WhiplashCopyFile` can solve this problem in a way that will keep
everybody happy.

Simply add your custom `.gitignore` file to the Whiplash
project-specific config directory for the `fubar` project. By default,
Whiplash looks for project config directories at
`~/projects/dotfiles/whiplash-config/`. So, Assuming a default Whiplash
configuration, you would create your custom `.gitignore` file at this
path:

    ~/projects/dotfiles/whiplash-config/fubar/.gitignore

Now, add the following line to the Whiplash `config.vim` file for the
`fubar` project:

    :WhiplashCopyFile .gitignore

From now on, whenever you invoke the `:Whiplash fubar` command, Whiplash
will automatically copy the custom `.gitignore` you created into this
location:

    ~/projects/fubar/.gitignore

This is a very powerful concept: team defaults with personal overrides.
Everybody receives a `.gitignore` file, but yours is always customized
to your taste. The only caveat is that you have to be careful not to
commit the overridden changes to `.gitignore` back to the team's shared
repo.

> Yes, I know that `.git/info/exclude` performs a similar function. But
> that file is not safely backed up in your project's Git repo. Your
> Whiplash config directory will likely be stored in a peronsal repo
> which you *will* be backing up regularly.

`:WhiplashCopyFile` is very useful for most dotfiles. I use it to inject
a customized `.agignore` into my projects.

You can also use `:WhiplashCopyFile` to copy files nested within
directories. The command below will copy `one/two/three.txt` into your
project directory, and create nested directories in the destination if
necessary.

    :WhiplashCopyFile one/two/three.txt

`:WhiplashCopyFile` currently only supports copying a single file. To
copy multiple files, call `:WhiplashCopyFile` multiple times! I am
planning to add support for copying entire directories and their
contents.

### :WhiplashEchoProjectName

The `:WhiplashEchoProjectName` command does exactly what it says on the
tin: it echos the name of the currently selected project.

Sometimes it's just reassuring to visually confirm which project you
selected.

## Protips

### Project Name Tab-Autocompletion

The `:Whiplash` invocation command supports tab-autocompletion of
project names. It will complete partially-entered project names if a
matching project is found in the directory specified by
`g:WhiplashProjectsDir`.

### Status Output

Call the `:Whiplash` command without a project name argument to see the
currently selected project and some related information. Here is what
the output looks like:

    Current Project : my-project
    Project Path    : ~/projects/my-project/
    Vim Working Dir : /Users/arkwright/projects/my-project

### Default Project

If you want `:Whiplash` to be called with a specific project name at the
moment Vim launches, add the following line to your `.vimrc` file.
Replace `project-name` with the name of your desired default project.

    autocmd VimEnter * Whiplash project-name

### Ignore Files

Often you will want Vim to ignore certain files in a project. `:set
wildignore` is Vim's built-in way of doing this. Add and customize
lines like these within your project's `config.vim`.

    set wildignore=
    set wildignore+=some/dir/you/want/to/ignore/
    set wildignore+=ignore_all_files_under_this_dir/**

> Command-T/ctrlp and other plugins use the `wildignore` option when
> filtering the results they display to you. A decent `wildignore` can
> speed up these plugins when working with projects containing large
> numbers of files.
> 
> The `set wildignore=` line clears your current `wildignore` setting so
> that you are starting from scratch when configuring `wildignore` for
> this project.

# License

[Unlicense](http://unlicense.org/)
