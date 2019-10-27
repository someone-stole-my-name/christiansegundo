---
layout: post
title: Get all followers or friends of a Twitter user
category: Programming
tags:
- perl
- twitter
- api
---

# What is the fastest way to get N number of followers from a big Twitter account

I faced this problem recently and the answer deppends on how big N is. There are basically two ways to fetch Twitter followers from an user:

 * Fetching full followers list using `followers/list`[[1]]. The rate limit with this approach is (at most) 200 * 15 = 3000 followers every 15 minutes.

 * Second method:
  * Fetching the followers ids using `followers/ids`[[2]]. The rate limit is 5000 * 15 = 75K follower ids every 15 minutes.
  * Then look up their usernames (or other data) using `users/lookup`[[3]]. Rate limit is about 100 * 90 = 90K every 15 minutes.

# Examples

All the examples are written in Perl and use `Twitter::API`[[4]] with `Echilada` and `RateLimiting` traits. Full versions available here[[5]].

## Example 1

This example uses the first method, it is very convinient for accounts with 3000 or less followers.

```perl
my $cursor = -1;

while($cursor != 0) {
    my $followers = $client->followers({
        'screen_name' => '****',
        'count' => 200,
        'cursor' => $cursor,
    });

    $cursor = $followers->{next_cursor};

    foreach my $user (@{$followers->{users}}) {
        print $user->{screen_name} . " : " . $user->{id_str} . "\n";
    }
}
```

## Example 2

This method uses two API calls,  it fetches 5000 followers and loop through them 100 at a time getting the details and repeating the steps as many times as necessary.

```perl
my $cursor = -1;

while($cursor != 0) {
	my $followers_ids = $client->followers_ids({
            'screen_name' => '****',
			'count' => 5000,
			'cursor' => $cursor,
		}
	);

	$cursor = $followers_ids->{next_cursor};

	for (my$i = 0; $i <= $#{$followers_ids->{ids}}; $i += 100) {
		my $id_100;
		foreach($i .. $i+100-1) {
			last unless @{$followers_ids->{ids}}[$_];
			$id_100 .= @{$followers_ids->{ids}}[$_];
			$id_100 .= ',' unless $_ == $i+100-1;
		}

		my $users = $client->lookup_users({'user_id'   => $id_100});
		foreach my $user (@{$users}) {
			print $user->{screen_name} . " : " . $user->{id_str} . "\n";
		}
	}
}
```
---
[1]: <https://developer.twitter.com/en/docs/accounts-and-users/follow-search-get-users/api-reference/get-followers-list> "GET followers/list"
[2]: <https://developer.twitter.com/en/docs/accounts-and-users/follow-search-get-users/api-reference/get-followers-ids> "GET followers/ids"
[3]: <https://developer.twitter.com/en/docs/accounts-and-users/follow-search-get-users/api-reference/get-users-lookup> "GET users/lookup"
[4]: <https://metacpan.org/pod/Twitter::API> "Twitter::API - A Twitter REST API library for Perl"
[5]: <https://github.com/someone-stole-my-name/Twitter_followers_examples> "Examples repository"