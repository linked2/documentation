# DRAFT DOCUMENTATION

Status: Unpublished

Author: Rob Smith

Version: 1.0

Date: 28th Dec 2018

# Linked2 iPaaS

Increasingly we don't use software applications in isolation, but rather as one tenant of a software platform. We use software as part of a cloud and we gain benefits from that.

The Linked2 platform integrates at a platform to platform level, providing an integration product specifically tailored for moving data between two specific platforms. Together with our white label approach this enables our integration products to appear to be part of the platforms they are integrating.

## Pipeline
Purely a descriptive term for a configured & live integration from source system to target system. In that sense the integration 'pipeline' includes code to send the data, the protocol its sent over, the receivers (webhook/api) that data for Linked2, the whole Linked2 process including publishing and the protocol used and finally the receiving api in the target system.

## Products

A Linked2 product is just like any other software product, it is basically just an uninstalled, unconfigured package that needs to be installed with specific configurations to be useful. You will have at least one Linked2 integration product.

## Stages

A stage is our term for an installed product. A stage is a live integration pipeline between two systems. Data only flows one way through a stage. At the time of writing we have only products with single stages, so to create a bi-directional integration would require installing two products. A future release will enable multiple stages in a single product making the installation process for more than single data flows much simpler.

## Filters

## Transformers

## Publishers & the Publish Context

## Support

## Webhooks and Batches

