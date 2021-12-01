# Create RPM repository on S3 storage

Using the traditional [createrepo](https://github.com/rpm-software-management/createrepo_c)
to create RPM repositories can be hard if you work with containers and S3
storage. Most solutions involve downloading the entire repository to update the
metadata. This can be costly when dealing with large repositories.

`createrepo-s3` is trying to solve this problem by allowing to add new files to
a repository living in S3 while keeping all old metadata and packages as they
are.

The script is basically a wrapper around `s3cmd` and `createrepo`.


## How it works

The tool needs to be run on a directory which only contains the RPMs you want to add.
It will then:

1. Use `s3cmd` to list all RPM files in S3
2. Touch all these files locally, creating an empty file for each RPM
3. Launch `createrepo`, instructing it to only check for file existence
4. Remove the empty files
5. Upload everything else to S3


## Prerequisites

- `createrepo` installed
- `s3cmd` installed and configured
    - Make sure `s3cmd ls s3://` (no arguments) works
- The S3 location must already contain a valid RPM repository.
  You can use `createrepo` for initialization:

        mkdir repository
        createrepo repository
        s3cmd put -r repository s3://my_bucket/


## Example

Add `xy-0.6.1-1.fc36.x86_64.rpm` to the repository:

```sh
mkdir repository
cp /path/tp/xy-0.6.1-1.fc36.x86_64.rpm repository/
createrepo-s3 repository/ s3://my_bucket/repository
```


## Limitations

- Bucket names and paths may not contain `#`
- This does not deal with concurrency. Simultaneous updates may overwrite each other.
- This only allows for adding new packages. Updating existing ones does not work.
  You should release a new version anyway ;-)
