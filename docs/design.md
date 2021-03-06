---
title: Mill Design Notes
markdown2extras: wiki-tables, code-friendly, cuddled-lists
---

# milld agent

    milld start
    milld stop
    milld restart
    milld status
    milld configtest

    milld add -t nginx /var/log/nginx.log
    milld add -t bunyan /var/log/myapp.log
    milld setup
        Interactively setup Manta info, and perhaps logs to handle.
        Q: setup a new SSH key for this? Probably want *one* key for multiple
        machines using the 'mill' role in Manta (once RBAC is there).
    milld config
        Like "git config ...".


# mill client tool

    mill cat service=NAME inst=ID [start=DATE] [end=DATE-OR-RANGE]
        Default start is 10m ago, default end=5m (i.e. "latest") ... kinda 'mill
        tail'.

    mill merge-cat [service=NAME1 service=NAME2 ...] [inst=ID1 inst=ID2...] [start=DATE] [end=DATE-OR-RANGE]
        Like `cat` but access multiple services and insdtances and will merge
        in time order.
        TODO: Add other filter predicates.

    mill grep [service=NAME1 service=NAME2 ...] [inst=ID1 inst=ID2...] start=DATE end=DATE-OR-RANGE SEARCH-TERM
        (Optionally ?) Does the merge on time if multiple service or inst.

    mill ls [service=NAME service=NAME2] inst=ID ...
        # all services
        # all instances
        # date ranges with logs for a service/instance
        This is sugar for now. So punting as a *req* for MVP.

    mill config
        Like "git config ...".


# mill Manta layout

    /$user/stor/mill/
        logs/
            $service/
                $year/$month/$day/$hour/
                    # Upload 5 minute chunks to allow working with smaller
                    # files for 'mill cat' etc. This might be a tunable at some
                    # point.
                    $inst-$timestamp.$type.log
        backfill/...    # TODO: dir to hold past logs to backfill into logs/...

Example:

    /trentm/stor/mill
        logs/
            webhead/
                2013/11/11/12/
                    # original files...
                    foo-1384294081423.apache.log
                    foo-1384294129834.apache.log
                    # ... are processed to
                    foo-1384294081423.apache.tlog   # "$timestamp," prefixed
                    foo-1384294081423.apache.ilog   # json-ified index
                    foo-1384294129834.apache.tlog
                    foo-1384294129834.apache.ilog
                    # ... then the original ".log" is deleted
        backfill/...    # TODO: or something for backdated logs to integrate


# Log types

Rules: Cannot have '.' or '-' in log type strings (else it screws up the
Manta mill layout).

Supported types:

- `generic`
- `apache`
- `bunyan`

TODO: perhaps use same naming as splunk (see Ben's PDF).
