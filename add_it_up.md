# Add It Up

Let

```
#!/usr/bin/env perl6

sub MAIN (Int $x, @numbers=@*ARGS) {
    my $total = $x;
    for @numbers -> $n {
        $total += $n;
    }
    put "Sum = $total";
}
```

