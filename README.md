# z.sh-bookmark
# !!! DEPRECATED !!!
use https://github.com/PEMessage/fasd-bookmark instead
## Introduce
A fork from rupa/z, but now you could bookmark a dir to get more control about the behavior!
Excpt `z` command, also provide a function `z-mark` allow you quickly bookmark a dir.


## Usage
```shell
# EXTRA USAGE (z.sh-bookmark)
#     * z -m                                  # show all bookmark (notice: it will overwrite others option)
#                                             # example : `z -mlt` will equal to `z -m` looking forward to improve in futrue
#     * z --read-mark [path or bookmark name] # if given a path return its bookmark name , given bookmark name return path
#     * z --mark [bookmark name] [path]       # show all bookmark 
#     * z [bookmark]                          # jump to bookmark, NOTICE: your will need to type full bookmark name
# HANDY FUNCTION (z-mark)
#     * z-mark [bookmark name]                # mark $PWD with [bookmark name]
#     * z-mark                                # clean $PWD bookmark
```

## fzf combine (z-fzf function)
1. source following script to your .bashrc or .zshrc or append it to z.sh
2. if you given any path as argument, it equal to z command 
   1. `cdd [path]` : equal to `z` , jump by mix rank and time
   2. `cdt [path]` : equal to `z -t`, jump by recent time
   3. `cdr [path]` : equal to `z -r`, jump by rank
3. if no argument was given, it will show list in fzf, press enter to `cd` to it
   1. `cdd` : sort list by mix time and rank
   2. `cdt` : sort list by time
   3. `cdr` : sort list by  rank
   4. `cdm` : show all bookmark

```shell
z-fzf(){ 

    _z_dirs(){
        [ -f "$datafile" ] || return

        local line
        while read line; do
            # only count directories
            [ -d "${line%%\|*}" ] && echo "$line"
        done < "$datafile"
        return 0
    }
    local datafile="${_Z_DATA:-$HOME/.z}"
    local fnd opt cmd
    while [ "$1" ]; do case "$1" in
        --) while [ "$1" ]; do shift; fnd="$fnd${fnd:+ }$1";done;;
        -*) opt=${1:1}; while [ "$opt" ]; do case ${opt:0:1} in
                t)cmd='t';;
                r)cmd='r';;
                m)cmd='m';;
                d)cmd='d';;
            esac; opt=${opt:1}; done;;
         *) fnd="$fnd${fnd:+ }$1";;
    esac; last=$1; [ "$#" -gt 0 ] && shift; done

    if [ "$cmd" = d ] ; then
        [ -n "$fnd" ] &&  _z  "$fnd" && return
        local temp=` _z -l 2>&1 | awk '{print $2}' | fzf  --no-sort | sed -e "s@^~@${HOME}@g" `
        cd "$temp"

    elif [ "$cmd" = t -o "$cmd" = r ] ; then
        [ -n "$fnd" ] &&  _z -"$cmd" "$fnd" && return
        local temp=` _z -l"$cmd" 2>&1 | awk '{print $2}' | fzf  --no-sort | sed -e "s@^~@${HOME}@g" `
        cd "$temp"
    elif [ "$cmd" = m ] ; then
        [ -n "$fnd" ] &&  _z "$fnd" && return
        local temp=` _z -m 2>&1 | fzf  --no-sort | awk '{print $2}' |  sed -e "s@^~@${HOME}@g" `
        cd "$temp"
    else
        return 1 
    fi
}

alias cdt='z-fzf -t '
alias cdr='z-fzf -r '
alias cdd='z-fzf -d '
alias cdm='z-fzf -m '
```

## Origin Document 
```man
Z(1)                             User Commands                            Z(1)



NAME
       z - jump around

SYNOPSIS
       z [-chlrtx] [regex1 regex2 ... regexn]

AVAILABILITY
       bash, zsh

DESCRIPTION
       Tracks your most used directories, based on 'frecency'.

       After  a  short  learning  phase, z will take you to the most 'frecent'
       directory that matches ALL of the regexes given on the command line, in
       order.

       For example, z foo bar would match /foo/bar but not /bar/foo.

OPTIONS
       -c     restrict matches to subdirectories of the current directory

       -e     echo the best match, don't cd

       -h     show a brief help message

       -l     list only

       -r     match by rank only

       -t     match by recent access only

       -x     remove the current directory from the datafile

EXAMPLES
       z foo         cd to most frecent dir matching foo

       z foo bar     cd to most frecent dir matching foo, then bar

       z -r foo      cd to highest ranked dir matching foo

       z -t foo      cd to most recently accessed dir matching foo

       z -l foo      list all dirs matching foo (by frecency)

NOTES
   Installation:
       Put something like this in your $HOME/.bashrc or $HOME/.zshrc:

              . /path/to/z.sh

       cd around for a while to build up the db.

       PROFIT!!

       Optionally:
              Set $_Z_CMD to change the command name (default z).
              Set $_Z_DATA to change the datafile (default $HOME/.z).
              Set  $_Z_MAX_SCORE  lower  to  age  entries  out faster (default
              9000).
              Set $_Z_NO_RESOLVE_SYMLINKS to prevent symlink resolution.
              Set $_Z_NO_PROMPT_COMMAND to handle PROMPT_COMMAND/precmd  your-
              self.
              Set $_Z_EXCLUDE_DIRS to an array of directory trees to  exclude.
              Set $_Z_OWNER to allow usage when in 'sudo -s' mode.
              (These  settings  should  go  in  .bashrc/.zshrc before the line
              added above.)
              Install the provided man page z.1  somewhere  in  your  MANPATH,
              like /usr/local/man/man1.

   Aging:
       The rank of directories maintained by z undergoes aging based on a sim-
       ple formula. The rank of each entry is incremented  every  time  it  is
       accessed.  When the sum of ranks is over 9000, all ranks are multiplied
       by 0.99. Entries with a rank lower than 1 are forgotten.

   Frecency:
       Frecency is a portmanteau of 'recent' and 'frequency'. It is a weighted
       rank  that depends on how often and how recently something occurred. As
       far as I know, Mozilla came up with the term.

       To z, a directory that has low ranking but has been  accessed  recently
       will  quickly  have  higher rank than a directory accessed frequently a
       long time ago.

       Frecency is determined at runtime.

   Common:
       When multiple directories match all queries, and they all have a common
       prefix, z will cd to the shortest matching directory, without regard to
       priority.  This has been in effect, if  undocumented,  for  quite  some
       time, but should probably be configurable or reconsidered.

   Tab Completion:
       z  supports tab completion. After any number of arguments, press TAB to
       complete on directories that match each argument. Due to limitations of
       the  completion  implementations,  only  the last argument will be com-
       pleted in the shell.

       Internally, z decides you've requested a completion if the  last  argu-
       ment  passed  is  an  absolute  path to an existing directory. This may
       cause unexpected behavior if the last argument to z begins with /.

ENVIRONMENT
       A function _z() is defined.

       The contents of the variable $_Z_CMD is aliased to _z 2>&1. If not set,
       $_Z_CMD defaults to z.

       The  environment  variable $_Z_DATA can be used to control the datafile
       location. If it is not defined, the location defaults to $HOME/.z.

       The environment variable $_Z_NO_RESOLVE_SYMLINKS can be set to  prevent
       resolving  of  symlinks.  If  it  is  not  set,  symbolic links will be
       resolved when added to the datafile.

       In bash, z appends a command to the PROMPT_COMMAND environment variable
       to maintain its database. In zsh, z appends a function _z_precmd to the
       precmd_functions array.

       The environment variable $_Z_NO_PROMPT_COMMAND can be set if  you  want
       to handle PROMPT_COMMAND or precmd yourself.

       The  environment  variable  $_Z_EXCLUDE_DIRS  can be set to an array of
       directory trees to exclude from tracking.  $HOME  is  always  excluded.
       Directories must be full paths without trailing slashes.

       The  environment  variable  $_Z_OWNER  can  be set to your username, to
       allow usage of z when your sudo environment keeps $HOME set.

FILES
       Data is stored in $HOME/.z. This  can  be  overridden  by  setting  the
       $_Z_DATA  environment variable. When initialized, z will raise an error
       if this path is a directory, and not function correctly.

       A man page (z.1) is provided.

SEE ALSO
       regex(7), pushd, popd, autojump, cdargs

       Please file bugs at https://github.com/rupa/z/



z                                January 2013                             Z(1)
```
