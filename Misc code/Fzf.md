
##### Fzf with preview

```
fzf --preview='cat {}'
```

##### Open fzf file in nvim

```
nvim $(fzf --preview='cat {}')
```