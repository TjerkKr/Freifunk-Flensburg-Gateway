
## Table of Contents

* [Requirements](#requirements)
* [Installation](#installation)
  * [Install a minimal Debian](#install-a-minimal-debian)
    * [B.A.T.M.A.N. and fastd](#b.a.t.m.a.n.-and-fastd)
    * [add repo.universe-factory.net repository](#add-repo.universe-factory.net-repository)
    * [downgrade to batman 14](#downgrade-to-batman-14)
    * [This is needed for any version](#this-is-needed-for-any-version)
* [Networking](#networking)
* [DHCP and DNS](#dhcp-and-dns)
  * [DHCP radvd IPv6](#dhcp-radvd-ipv6)
  * [DHCP isc-dhcp-server IPv4 and IPv6](#dhcp-isc-dhcp-server-ipv4-and-ipv6)
  * [DNS bind9](#dns-bind9)
* [VPN](#vpn)

## Requirements

You should only need to update the global installation of `create-react-native-app` very rarely, ideally never.

Updating the `react-native-scripts` dependency of your app should be as simple as bumping the version number in `package.json` and reinstalling your project's dependencies.

Upgrading to a new version of React Native requires updating the `react-native`, `react`, and `expo` package versions, and setting the correct `sdkVersion` in `app.json`. See the [versioning guide](https://github.com/react-community/create-react-native-app/blob/master/VERSIONS.md) for up-to-date information about package version compatibility.

## Installation

If yarn was installed when the project was initialized, then dependencies will have been installed via yarn, and you should probably use it to run these commands as well. Unlike dependency installation, command running syntax is identical for yarn and npm at the time of this writing.

## Install a minimal Debian

Runs your app in development mode.

Open it in the [Expo app](https://expo.io) on your phone to view it. It will reload if you save edits to your files, and you will see build errors and logs in the terminal.

## B.A.T.M.A.N. and fastd

Runs the [jest](https://github.com/facebook/jest) test runner on your tests.

## add repo.universe-factory.net repository

Like `npm start`, but also attempts to open your app in the iOS Simulator if you're on a Mac and have it installed.

## downgrade to batman 14

Like `npm start`, but also attempts to open your app on a connected Android device or emulator. Requires an installation of Android build tools (see [React Native docs](https://facebook.github.io/react-native/docs/getting-started.html) for detailed setup).

## This is needed for any version

This will start the process of "ejecting" from Create React Native App's build scripts. You'll be asked a couple of questions about how you'd like to build your project.

## Networking

## DHCP and DNS

## DHCP radvd IPv6

## DHCP isc-dhcp-server IPv4 and IPv6

## DNS bind9

## VPN
