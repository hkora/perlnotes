---
layout: post
tags: [perl, testing]
comments: True
title: perl习语（idioms）
---

_There's more than one way to do it._

__The notes are based on "Idiomatic Perl" - Dave Cross.__

在使用perl完成了一个测试框架和看了不少资料后，留下了一些笔记。

1. `||`和`or`的优先级不同

    `||`优先级高，`or`优先级低，这造成了很多问题。

        $a = $b or $c; #这样用or有问题
        ($a = $b) or $c; #实际得到
        $a = $b || $c; #应该用||


        open FILE, $file || die("open $file fail"); #这样用||有问题
        open FILE, ($file || die("open $file fail")); #实际得到
        open FILE, $file or die("open $file fail"); #应该用or

2. 设定默认值

        my $time = shift || "0:0:0";

    潜在问题是shift出来的是0或者''，而且0或者''是希望得到的值。
    因为这时$time会得到"0:0:0"。

    perl5.10引入了//解决这个问题。

        my $time = shift // "0:0:0";

    这时$time会得到希望的0或者''。

3. 写文件

        sub write_file
        {
          my ($filename, $content) = @_;
          open (my $wfh, '>', "$filename") || croak("cannot write file $filename");
          for (@{$content}) {
            print $wfh $_;
          }
          # close tsc file
          close $wfh;
        }

4. use Carp;

    使用Carp的好处确实很多。它可以得到message从哪里抛出来的。
    carp, cluck, croak, confess的区别主要如下：

                 Fatal  Backtrace
        carp       N        N
        cluck      N        Y
        croak      Y        N
        confess    Y        Y

5. 用来计算成功，失败运行的个数

        my %test_status = 
        (
            total_dif => \@dif_tests,
            total_suc => \@suc_tests,
            ...
        )
        my $totalsuc = scalar @{$test_status->{total_suc}};
        ...

6. 用来自定义排序

        sub sort_custom
        {
            my ($array, $wfh, $compare_method) = @_;
            if ($array)
            {
                foreach (sort $compare_method @$array) 
                {
                    ...
                }
            }
        }
        sort_custom($test_status->{total_tests}, $result_filename, \&compare_by_testname);
        sub compare_by_testname
        {
          my @ta = split('\t',$a);
          my @tb = split('\t',$b);
          my $testnamea = $ta[0];
          my $testnameb = $tb[0];
          $testnamea cmp $testnameb;
        }

7. 用map和grep来取不重复的元素

        sub uniq_test
        {
          my $arr = shift;
          my %temp_hash;
          my @result = grep { ! $temp_hash{(split('\t', $_))[0]}++ } map lc, @{$arr};
          return \@result;
        }

8. 创建dispatch，根据不同参数调用不同的程序 

        my %arg_dispatch_table =
        (
          component => \&p_component,
          testname => \&p_testname,
          ...
        )

9. 将hash改造成set用

        %humped = 
        {
            camel     => 1,
            dromedary => 1,
        };
        $humped{$animal} or die "$animal has no humps\n";

10. my vs local

    my和local的区别，在网上这个程序演示很棒。

    如果$var是local，得到的结果是4 10 11 4

    如果$var是my，得到的结果是4 10 10 5

        $var = 4;
        print $var, "\n";
        hello();
        print $var, "\n";
        # subroutines
        sub hello {
             local $var = 10;    # change here for testing
             print $var, "\n";
             gogo(); # calling subroutine gogo
             print $var, "\n";
        }
        sub gogo {
             $var ++;
        }

11. perl会把array flatten

        @arr_2d = ((1, 2, 3),(4, 5, 6),(7, 8, 9));

    @arr_2d 得到的是 (1, 2, 3, 4, 5, 6, 7, 8, 9)

12. 如果要创建2维的array

        @arr_2d = ([1, 2, 3],[4, 5, 6],[7, 8, 9]);

    $arr_2d[1] 是 (4, 5, 6)

    $arr_2d[1]->[1] 是 5


13. The Schwartzian Transform 这个施瓦茨变换太棒了。

        @data_out =
         map { $_->[1] }
         sort { $a->[0] cmp $a->[0] }
         map { [func($_), $_] } @data_in;

14. 很多程序里面会省略这个默认变量$_

15. 默认的记录分隔变量$/

16. 一口吃下整个文件

        $/ = undef;

17. 相当于使用'paragraph'模式.

        $/ = ''; # empty string

18. 打印当前记录号$.

        while (<FILE>) {
            print "$.: $_";
        }

19. $,是输出field分隔符, 默认值是empty string

        @nums = 1 .. 3;
        print @nums; # 123
        $, = ' - ';
        print @nums; # 1 - 2 - 3

20. $"是list分隔符, 默认值是space

        @nums = 1 .. 3;
        print "@nums"; # 1 2 3
        $" = '|';
        print "@nums"; # 1|2|3

21. $!存储了最后一个os调用的错误 

        open FILE, 'file' or die $!;

22. $?存储了最后一个子进程的错误

        @file = `ls`;
        die $? if $?;

23. $@存储了最后一个"eval"调用的错误

        eval { require Module };
        die $@ if $@;

24. $0当前程序名

25. $$当前进程号

26. $^O存储当前操作系统

27. %ENV存储所有环境变量

28. matched patterns，$1, $2...

        $_ = 'This is a string';
        if (/is (\w+) (\w+)/) {
         print "$1 : $2\n";
        }

29. grep

    grep作用在每个元素，每个元素记录在$_，如果符合{}中的判断，才加入到输出列表

        @odds = grep { $_ % 2 } @ints;

30. map

    map作用在每个元素，每个元素记录在$_，所有元素经过{}中的处理，都加入到输出列表

        @squares = map { $_ * $_} @ints;

31. 用qq和q方便的表示双引号和单引号


        print "<img src=\"$file\" width=\"100\" height=\"50\">";
        print qq(<img src="$file" width="100" height="50">);
        q!some text!
        @days = ('Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat');
        @days = qw(Sun Mon Tue Wed Thu Fri Sat);


