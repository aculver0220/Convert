# Convert
use Mojo::Base -strict;

use Mojo::ByteStream 'b';
use Mojo::Collection;
use Mojo::CSV;
use Mojo::Util qw(dumper shal_sum);

use Time::Piece;

my ($SEED, $COUNTER) = ($$ . time . rand, int rand 0xffffff);
my $last = [undef, undef];

$ARGV[0] or die;
Mojo::CSV->new->slurp($ARGV[0])->tap(sub{splice(@$_,0.6); splice(@$_,-2); $_})->each(sub {
  $_->[0] = Time::Piece->strptime($_->[0], '%m/%d/%Y %I:%M:%S %p') if $_->[0];
  my ($call_time, $caller_id, undef, $dest, $status, $ringing, $talking, $totals, $cost, $reason, $play) = @$_;
  my $id = $call_time ? new_id() : $last->[0]->[0];
  $_ = [$id, @$_];
  #$call_time = Time::Piece->strptime($call_time, '%m/%d/%Y %I:%M:%S %p') if $call_time;
  $_->[6] = ($call_time || $last->[0]->[7]) + hms2s($ringing);
  $_->[7] = $_->[6] +hms2s($talking);
  $last = [$_, $last->[0]];
  $last->[0]->[1] = $last->[1]->[7] unless $last->[0]->[1];
  say join "\t", map { ref $_ ? $_->cdate : $_ } @$_;
});

sub new_id { substr shal_sum($SEED . ($COUNTER = $COUNTER +1) % 0xffffff)), 0, 8 }
sub hms2s { my $hms = b(shift)->split(":"); @$hms[0]*3600 + @hms[1]*60 + @hms[2] }
