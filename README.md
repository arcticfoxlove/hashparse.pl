hashparse.pl
============

parse blast output to get only qid sid and score


#!/usr/bin/perl
#hashparse.pl parse blast output with hashes- key1 is query, key2 is search, value is score 
#priyanka kulkarni
#nov 19, 2014
#usage statement: ./hashparse.pl blast_output

use strict; use warnings;

my ($input) = @ARGV;
open (my $in, "<", $input) or die "error reading ARGV[0] for reading";
open (my $out, ">", "$input.result") or die "error writing to file";

my $query = "";
my %data;
my $count =0;

while (<$in>) {
	next until $_ =~ m/chr/;
	if ($_ !~ m/>/) {
		chomp;
		my $line = $_;
		$line =~ s/\s{2,80}/\t/g;    #if there are more than 2 spaces change those into tab
		my @list = split("\t", $line);
		if ($list[1] =~ m/[a-z]/){    #if list[1] has alphabet
			$query = $list[1];
		}
		elsif ($list[1] >= 50){		#if list[1] is a number (score) greater than 20
			if ($query ne $list[0]){
				$data{$query}{$list[0]} = $list[1];		#@hash{key1}{key2} = value;
			}
		}
	}
}
close $in;

foreach my $qid (keys %data) {#for each my queryid in hash, print query
	print $out("$qid\n");
    $count++;
	foreach my $sid (keys %{$data{$qid}}) {		#for each of my searchids in hash{key1}
	print $out("\t$sid\t$data{$qid}{$sid}\n");	#print search, and value
	}	
}

close $out;

print "#NEXUS\n\n\n";
print "BEGIN taxa;\n";
print "\tDIMENSIONS ntax= $count\n";
print "TAXLABELS\n";
foreach my $qid (keys %data){
    print "\t$qid\n";
}
print ";\nEND;\n\n\n";
print "BEGIN distances;\n\tDIMENSIONS ntax=$count;\n\tFORMAT\n\t\ttriangle=LOWER\n\t\tdiagonal\n\t\tlabels\n\t\tmissing=?\n\t;\n\tMATRIX\n";


__END__
