---
layout: post
title: perl 影像辨識 影像比較
date: 2012-04-16
tags: perl Compare
---
perl 影像辨識 影像比較

安裝

```
apt-get install libpng-dev
perl -MCPAN -e 'install Image::Compare' 
perl -MCPAN -e 'install Imager::File::PNG';
sudo apt-get -y install libgd2-xpm-dev build-essential
perl -MCPAN -e 'install GD'
```

perl 影像辨識 影像比較程式

```
use Image::Compare;

 my($cmp) = Image::Compare->new();
 $cmp->set_image1(
     img  => '/home/chio/snap-h264-9.png',
     type => 'png',
 );
 $cmp->set_image2(
     img  => '/home/chio/snap-h264-6.png',
     type => 'png',
 );
 $cmp->set_method(
     method => &Image::Compare::THRESHOLD,
     args   => 25,
 );
 if ($cmp->compare()) {
     print "The images are the same, within the threshold \n";
 }
 else {
     print "The images differ beyond the threshold \n"; }
```

後來改成 灰階後比較 ....

perl g.pl file1.png fil2.png

```
#! perl -slw
use strict;
use GD;

GD::Image->trueColor( 1 );

my $im1 = GD::Image->new( $ARGV[ 0 ] );
my $im2 = GD::Image->new( $ARGV[ 1 ] );

my $raw1 = $im1->gd;
my $raw2 = $im2->gd;

my $xored = $raw1 ^ $raw2;
my( $all, $diff ) = (0)x2;

$all += 255, $diff += ord substr $xored, $_, 1 for 0 .. length( $xored ) - 1;

print $all, ' ', $diff;

printf "The simlarity is %.3f%%\n", ( $all - $diff ) / $all * 100;
```
