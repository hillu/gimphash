gimphash
--------

gimphash is a proposed method to calculate an [imphash](https://www.mandiant.com/resources/tracking-malware-import-hashing) equivalent for [Go](https://go.dev/) binaries.

Golang binaries contain their dependencies as part of the executable. These dependencies include both standard library packages and third party dependencies and can be used, analogous to a classical imphash, to identify a Golang project.

The dependencies can be listed using the [pclntab](https://go.dev/src/debug/gosym/pclntab.go) that is part of each Golang binary (also see this [blog post](https://www.mandiant.com/resources/golang-internals-symbol-recovery) by Mandiant). The pclntab contains a number of interesting elements for reverse engineering; for the gimphash we will use the function names that are contained there.

Calculation
-----------
1. Locate the pclntab within a Golang binary
2. Enumerate golang functions using the functab within the pclntab:
    1. Ignore functions starting with `go.` or `type.` (compile artefacts, runtime internals)
    2. Ignore functions starting with `internal/`  or `vendor/` ('vendoring' of the standard library)
    3. Reduce the function name to the part before the first `.` that is after the last `/` (e.g. `golang.org/x/sys/windows.CloseHandle` becomes `golang.org/x/sys/windows`)
    4. If the part before the first `/` contains a `.` and is NOT in the following list, ignore the function name: (ignoring private repositories; often serve as source code instead of 'imports' that we'd like to hash here)
        - `golang.org`
        - `github.com`
        - `gitlab.com`
        - `gopkg.in`
        - `google.golang.org`
        - `cloud.google.com`
3. Sort the resulting package names (alphabetically) 
4. Calculate the SHA-256 hash over the concatenated names (no delimiter)

Feedback
--------

This specification and the related code are a draft. Please use the "issues" 

Step 2 IV
=========
As an alternative to the step 2 iv, we could identify the filepath of the main module and use this to exclude packages that are part of the built project. Feedback on whether this might be better than the current whitelist approach is appreciated.

Step 3
======
Alternatively we could skip sorting the package names. (see this [comment](https://twitter.com/invisig0th/status/1526207532741664769) on Twitter)

