# vsgCollective

Project that collects all the main VulkanSceneGraph projects together under one repository/directory structure with build support.

To checkout:

    git clone https://github.com/robertosfield/vsgCollective.git

To build:

    cmake .
    make -j 8

This will download/checkout the required external projects and build each project and place them all in local include and lib directories.
