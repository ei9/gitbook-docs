# I. The Source Code Repository

The PostgreSQL source code is stored and managed using the Git version control system. A public mirror of the master repository is available; it is updated within a minute of any change to the master repository.

Our wiki, [https://wiki.postgresql.org/wiki/Working\_with\_Git](https://wiki.postgresql.org/wiki/Working\_with\_Git), has some discussion on working with Git.

Note that building PostgreSQL from the source repository requires reasonably up-to-date versions of bison, flex, and Perl. These tools are not needed to build from a distribution tarball, because the files that these tools are used to build are included in the tarball. Other tool requirements are the same as shown in [Section 16.2](https://www.postgresql.org/docs/10/static/install-requirements.html).
