---
title: Creation of Trove-Compatible Images for RDO
date: 2015-03-30 10:41:11
author: rbowen
---

[A guest post by Victoria Martínez de la Cruz] 

OpenStack Trove, included in the latest RDO releases, allows users to use the features of relational and non-relational databases without the added complexity of handling administrative tasks. With Trove, database users don't need to be database experts to provision and manage multiple databases instances.

Apart from the installation and configuration of the different modules you require to use Trove, you will need to build an image specific to the OS and storage backend of your choice.

Currently, there is no easy way to build these images: you can either inject the required files manually or you can use an utility called diskimage-builder.

In this guide we are going to explain how to build a RHEL-based Trove compatible images using a script that wraps diskimage-builder and adds all the required bits.

Red Hat Enterprise Linux (RHEL) 7 is the minimum recommended version, or the equivalent version of one of the RHEL-based Linux distributions such as CentOS 7, or Fedora 21 or later, to do this task. Also, we assume you have the latest RDO, and that you have already subscribed to the appropriate repositories.

1. Install trove-image-elements
===============================

The trove-image-elements repository contains the code necessary to build the images with the supported storage backends -- currently MySQL and MongoDB. Install git and clone it in your environment using the following directives:

> $ sudo yum -y install git
> $ sudo git clone https://github.com/vkmc/trove-image-elements

2. Download a RHEL-based guest image
====================================

Get a base guest image for the distro of your choice. You can get RHEL 7, CentOS 7 and Fedora 21 guest images in https://openstack.redhat.com/Image_resources. You can also get images for RHEL 6, CentOS 6 or Fedora 20, but the image creation process has not been tested with those versions.

3. Create the image
===================

The diskimage-builder tool works by taking a disk image, installing pieces of software - called 'elements' - and repacking it for use.

Running the script will generate a (a great deal of) log output, as the utility opens your base image, registers it to RHSM (in the case of RHEL), updates the system image, installs trove-guestagent and the storage backend of your choice, and packages the image for use in Trove. At the end, you'll have a file named ``DISTRO-DATASTORE-guest-image.qcow2``, where DISTRO and DATASTORE are the GNU/Linux distribution and storage backend you selected.

3.1 Create a RHEL7 image
------------------------

Run create_trove_image.sh with the following parameters to create a RHEL 7 image with MySQL:

> $ sudo ./create_trove_image.sh --distro rhel --datastore mysql --local-image &lt;path-to-local-image&gt; --rh-user <your-rh-user> --rh-password &lt;your-rh-password&gt; --rh-pool-id &lt;subscription-pool-id&gt;

or 

> $ sudo ./create_trove_image.sh -d rhel -s mysql -i &lt;path-to-local-image&gt; -u &lt;your-rh-user&gt; -p &lt;your-rh-password&gt; -o <subscription-pool-id>

3.2 Create a CentOS 7 image
---------------------------

Run create_trove_image.sh with the following parameters to create a CentOS 7 image with MySQL:

> $ sudo ./create_trove_image.sh --distro centos --datastore mysql --local-image &lt;path-to-local-image&gt;

or

> $ sudo ./create_trove_image.sh -d centos -s mysql -i &lt;path-to-local-image&gt;

3.3 Create a Fedora 21 image
----------------------------

Run create_trove_image.sh with the following parameters to create a Fedora 21 image with MySQL:

> $ sudo ./create_trove_image.sh --distro fedora --datastore mysql --local-image &lt;path-to-local-image&gt;

or

> $ sudo ./create_trove_image.sh -d fedora -s mysql -i <path-to-local-image>

4. Register the Image with Trove
================================

First, upload the image to Glance, using either the UI or the API. In the
UI, you can find this in your project's interface at Compute/Images.

Second, register this image with Trove. There is no way to do this from the UI yet, so
you have to do this using the API. For this, we provide a second script load_trove_image.sh.

Simply run load_image_trove.sh specifying the datastore, the datastore version, the packages
required by the image you want to upload and the image id generated by Glance. 

> $ sudo ./load_trove_image.sh --datastore mysql --datastore-version centos-mysql5.5 --packages mysql-server=5.5 --id bbd73560-58aa-4377-b961-3d12e76b0bed

or

> $ sudo ./load_trove_image.sh -s mysql -v centos-mysql5.5 -p mysql-server=5.5 -i bbd73560-58aa-4377-b961-3d12e76b0bed

You are now ready to begin using Trove: launch a new instance using the image you just created and start creating your DBs.