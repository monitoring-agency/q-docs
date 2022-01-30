# Variables

Variables are a great way to minify the effort to configure lots of similar systems. 

## Variable inheritance
As there are many places to put variables, following inheritance is used
if there are multiple variables with the same key:

Let's assume we have the following objects:

| Object name |   Object type   |     Variable     |
|:-----------:|:---------------:|:----------------:|
|    `gv`     | Global variable | `ssh_port: 2222` |
|    `ht`     |  Host template  | `ssh_port: 2323` |
|     `h`     |      Host       | `ssh_port: 3232` |
|    `mt`     | Metric template |  `ssh_port: 22`  |
|     `m`     |     Metric      | `ssh_port: 222`  |

with the following relations:

- `h` -> `ht`
- `m` -> `mt`

So if we have a check linked to `m` with the following syntax:
```
echo $ssh_port$
```

the output would be `222`.

**Let's break that down**

As variables are not bound to any context other than its inheritance, 
it's pretty obvious global variables getting overwritten by all the other ones.

As template inheritance is executed before the check is invoked, we can ignore the variables
of `ht` and `mt` as their variables got overwritten by `h` / `m`.

As the check is linked to `m`, the variable of `h` is not used as `m` holds a variable
with the same name. So the output would be `222` in the end.

If the check would be linked to `h`, the output would be `22` as the host never queries
the variables of its metrics.

