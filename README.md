Cron puppet module
==================

About
-----

This is a generic cron module for puppet.

#### init.pp

This contains the main cron class. It can be passed `$hiera_hash` and `$purge`
parameters which should both be booleans, see below for details.
The other classes are also included from this so you can simply `include cron`.

### crontab class

The parameter `$jobs` is taken and wrapped with the `create_resources` function
as such this simply takes cron jobs with the standard puppet cron type
attributes.

There is also a `$defaults` parameter intended to minimise boiler plate. This
enables default attributes to be set once and applied for all puppet defined
cron jobs. In this class this is set to ensure the jobs is enabled and that
root is the user to execute the jobs.

### interval class

These are similar to the crontab class but differ in `$jobs` should only provide
the contents of the job and the interval at which to run.

Note: Creating jobs via parameters for these classes requires `parser = future`.

This class also supports pulling files from the files directory of the module
itself on the puppet master. For example placing a file in `files/hourly` would
result in the client picking up that file in `/etc/cron.hourly/`.

### hiera_hash

If the `$heria_hash` parameter is true values for `$jobs` across all hierarchies
are used so cron jobs can be defined at multiple levels. This allows for a base
set of cron jobs to be defined at a low level of the hierarchy and more specific
cron jobs to be defined further up the hierarchy closer to what is likely more
relevant definitions.
This is definable in each of the classes expect in the service class. If defined
in the main cron class as true it will take precedence over definitions in other
classes.

### purge

All classes with the exception of the service class take a `$purge` parameter
which if set to true will cause puppet to remove any cron jobs that are not
provided by puppet. If defined in the main cron class as true it will take
precedence over definitions in other classes.

Usage
-----

A basic example of adding some cron jobs in YAML.

```yaml
# Add two cron jobs to crontab

cron::crontab::jobs:
  first_job:
    command:  '/bin/echo "This is run as root every 12 hours"'
    hour:     '*/12'
  second_job:
    command:  '/bin/echo "This is run as puppet every 12 hours"'
    hour:     '*/12'
    user:     'puppet'

# Add job to /etc/cron.daily/

cron::interval::jobs:
  once_a_day:
    command:  '/bin/echo "Today is $(/bin/date)"'
    interval: 'daily'

# Only allow puppet crontrolled jobs anywhere on the system

cron::purge: true

# Only allow puppet controlled jobs in crontabs ( e.g. /var/spool/cron/$user)

cron::crontab::pruge: true

# Only allow puppet controlled jobs in the periodic cron directories such as
# /etc/cron.hourly, /etc/cron.daily, /etc/cron.weekly and /etc/cron.monthly

cron::interval::pruge: true
```

Dependencies
------------

This module depends upon `puppetlabs/stdlib` for validation checks.

