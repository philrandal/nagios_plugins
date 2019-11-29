#!/usr/bin/perl -w

# based VERY loosely on a script called "check_freeradius.pl" found without a license at
# https://exchange.nagios.org/directory/Plugins/*-Plugin-Development-Tools/check_freeradius-2Epl/details


use POSIX;
use strict;
use Getopt::Long qw(:config no_ignore_case);
use Time::HiRes qw(gettimeofday tv_interval);

my $script = "check_freeradius.pl";
my $version = "2.0";

# default values
my $request_type;
my $host = "localhost";
my $port = 1812;
my $ipmode = 4;
my $timeout = 10;
my $secret = "";
my %avpairs = ("Message-Authenticator" => "0x00");
my $radclient_binary = "/usr/bin/radclient";
my $retries = 1;
my $send_count = 1;
my $warn_thresh = 3;
my $crit_thresh = 7;
my $debug = 0;

GetOptions (
    "f|function=s" => \$request_type,
    "H|host:s" => \$host,
    "p|port:1812" => \$port,
    "6|ipv6" => sub{ $ipmode = 6 },
    "t|timeout:8" => \$timeout,
    "s|secret:s" => \$secret,
    "a|avpair:s" => \%avpairs,
    "c|client:s" => \$radclient_binary,
    "r|retries:1" => \$retries,
    "u|count:1" => \$send_count,
    "W|warn:3" => \$warn_thresh,
    "C|crit:7" => \$crit_thresh,
    "d|debug" => \$debug,
    "V|version" => sub{ printf("$version\n"); exit 1; },
    "help" => sub{ show_usage(); },
);

if (!$request_type) {
    show_usage(2);
}

###################################
### Other Variables here    ###
###################################
my $command = "";
my $t0;
my $elapsed;
my @avstrings;
my $avstring;
my $status;

my %ERRORS = ('OK'=>0,'WARNING'=>1,'CRITICAL'=>2,'UNKNOWN'=>3,'DEPENDENT'=>4);

while (my($key, $val) = each %avpairs) {
    push(@avstrings, "$key=$val");
}
$avstring = join(",", @avstrings);

if ( $request_type eq "auth" || $request_type eq "acct" || $request_type eq "status") {
    $command = ("printf '%q' '$avstring' | $radclient_binary -$ipmode -q -c $send_count -r $retries -t $timeout $host:$port $request_type $secret");
    if ($debug) {
        printf("DEBUG: Using config request_type = $request_type, host = $host:$port (IPv$ipmode), timeout = $timeout, secret = $secret, avpair = \"$avstring\", radclient_binary = $radclient_binary, warn_thresh = $warn_thresh, crit_thresh = $crit_thresh, debug = $debug\n");
        printf("DEBUG: radclient command to send: $command\n");
    }
} else {
    printf("ERROR: Unknown functions, please read help for instructions on how to use this program");
    show_usage(2);
}

$t0 = [gettimeofday];
system($command);
$elapsed = tv_interval($t0);

if ( ($elapsed >= $crit_thresh) || ($? !=0) ) {
    $status = $ERRORS{'CRITICAL'};
    printf("CRITICAL: Radius response time: $elapsed secs, warning threshold: $warn_thresh, critical threshold: $crit_thresh, radclient exit status: $?, $script exit STATUS: $status.");
    exit $status;
} elsif ( ($elapsed < $warn_thresh ) ) {
    $status = $ERRORS{'OK'};
    printf("OK: Radius response time: $elapsed secs, warning threshold: $warn_thresh, critical threshold: $crit_thresh, radclient exit status: $?, $script exit STATUS: $status.");
    exit $status;
} elsif ( ($elapsed >= $warn_thresh) ) {
    $status = $ERRORS{'WARNING'};
    printf("WARNING: Radius response time: $elapsed secs, warning threshold: $warn_thresh, critical threshold: $crit_thresh, radclient exit status: $?, $script exit STATUS: $status.");
    exit $status;
} else {
    $status = $ERRORS{'UNKNOWN'};
    printf("CRITICAL: Radius response time: $elapsed secs, warning threshold: $warn_thresh, critical threshold: $crit_thresh, radclient exit status: $?, $script exit STATUS: $status.");
    exit $status;
}

sub show_usage
{
    print << "USAGE"; 
\n$script $version
Usage: $script [OPTIONS]

 Operational mode:
    -f, --function=request_type

 request_type is mandatory and must be one of the following:
    auth                    test authentication (Access-Request)
    acct                    test accounting (Accounting-Request)
    stat                    test server status (Server-Status)

 Server options:
    -H, --host=SERVER       connect to SERVER (default localhost)
    -p, --port=port         connect to UDP port port (default 1812)
    -6, --ipv6              connect using IPv6
    -s, --secret=SECRET     use shared secret SECRET (default is empty string)
    -t, --timeout=SECS      timeout after SECS seconds (default 10)
    -r, --retries=NUM       retry NUM times on failure or timeout (default 1)
    -u, --count=NUM         send each packet NUM times (default 1)

 Request options:
    -a, --avpair=ATTR=VAL   define attribute/value pairs for the request
                            (default Message-Authenticator=0x00)

 Other options:
    -c, --client=CLIENT     path to radclient if needed
    -W, --warn=SECS         return warning if request takes more than SECS s
                            (default 3)
    -C, --critical=SECS     return critical if request takes more than SECS s
                            (default 7)
    -h, --help              show usage information
    -d, --debug             enable debugging output

Examples:

    ./$script -f auth -host 10.10.10.1 -port 1812 -t 8 -s testing123 \
        -a User-Name=alex -a NAS-Port-Id=pw-285 -a NAS-IP-Address=10.10.10.1 \
        -c /usr/local/freeradius/bin/radclient -W 5 -C 10 -d

    ./$script --function status --host 192.168.34.2 --port 1812 --timeout 8 \
        --secret testing123 --client /usr/local/bin/radclient --crit 8 --debug

    ./$script --help

USAGE
    my ($err) = @_;
    $err ||= 1;
    exit $err;
}

