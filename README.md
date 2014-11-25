#!/usr/bin/perl
#hashparse.pl parse blast output with hashes- key1 is query, key2 is search, value is score 
#priyanka kulkarni
#nov 19, 2014
#usage statement: ./hashparse.pl blast_output

use strict; use warnings;

my ($input) = @ARGV;

open (my $in, "<", $input) or die "error reading ARGV[0] for reading";
open (my $out, ">", "$input.result") or die "error writing to file";
open (my $out2, ">", "$input.nexus") or die "error writing to nexus file"; #nexus format

my $query = "";
my %data;
my $count =0;
my $temp= "";

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
		elsif ($list[1] >= 300){		#if list[1] is a number (score) greater than 50
				$data{$query}{$list[0]} = $list[1];		#@hash{key1}{key2} = value;		
		}
	}
}
close $in;

foreach my $qid (keys %data) {#for each my queryid in hash, print query
	if (scalar keys %{$data{$qid}} > 1){
		print $out("$qid\n");
    	$count++;
    	
		foreach my $sid (keys %{$data{$qid}}) {		#for each of my searchids in hash{key1}
		print $out("\t$sid\t$data{$qid}{$sid}\n");	#print search, and value
	}	
	}
}
close $out;

#nexus format
print $out2("#NEXUS\n\n\nBEGIN taxa;\n\tDIMENSIONS ntax= $count\nTAXLABELS\n");
foreach my $qid (keys %data){
	if (scalar keys %{$data{$qid}} > 1){
    	print $out2("\t$qid\n");
	}
}
print $out2(";\nEND;\n\n\nBEGIN distances;\n\tDIMENSIONS ntax=$count");
print $out2(";");
print $out2("\n\tFORMAT\n\t\ttriangle=LOWER\n\t\tdiagonal\n\t\tlabels\n\t\tmissing=?\n\t;\n\tMATRIX\n");

my @array_qid;
foreach my $qid (keys %data){
	if (scalar keys %{$data{$qid}} >1){
		print $out2("\t$qid\t");
		push (@array_qid, $qid);
	
		for (my $i=0; $i< @array_qid; $i++){  # to get the size of an array, just $i< @array_qid
			if ( defined $data{$qid}{$array_qid[$i]} ) {
				my $new = (1/ $data{$qid}{$array_qid[$i]}); 
				printf $out2("%.5f\t" , $new);
			}
			else{
				print $out2("1.00\t");
			}
		}
		print $out2("\n");
	}

}
print $out2("\t;\n");
print $out2("END;");

close $out2;

__END__

