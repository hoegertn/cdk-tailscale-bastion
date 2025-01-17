# cdk-tailscale-bastion

![GitHub Workflow Status](https://img.shields.io/github/workflow/status/Hawxy/cdk-tailscale-bastion/release?label=Build%20%26%20Release&style=flat-square)
[![npm](https://img.shields.io/npm/v/cdk-tailscale-bastion?style=flat-square)](https://www.npmjs.com/package/cdk-tailscale-bastion)

This packages creates an AWS Bastion configured for Tailscale. This covers steps 1,2 & 4 of the [Tailscale RDS guide](https://tailscale.com/kb/1141/aws-rds/).

You may find this package useful if you need high performance access to internal resources (ie RDS) without the overhead & limitations of Session Manager.

## Installation

`npm i cdk-tailscale-bastion -D`

## Instructions

The Tailscale Auth key should be passed in via secrets manager and NOT hardcoded in your application. 

```typescript
import { TailscaleBastion } from 'cdk-tailscale-bastion';

// Secrets Manager
const secret = Secret.fromSecretNameV2(stack, 'ApiSecrets', 'tailscale').secretValueFromJson('AUTH_KEY');


const bastion = new TailscaleBastion(stack, 'Sample-Bastion', {
  vpc,
  tailscaleCredentials: {
    secretsManager: {
      secret: secret,
      key: 'AUTH_KEY',
    },
  },
});

// Remember to allow the host to read the secret!
secret.grantRead(bastion.bastion);
```

Whatever resource you intend to reach should permit connections from the bastion on the relevant port, naturally. 

## Tailscale Auth Key

I recommend generating an Ephemeral key that includes the bastion as a tag for ease of teardown and tracking:

<img src="https://user-images.githubusercontent.com/975824/177150876-ab21b4ac-00f7-4a75-befa-cf7d2e9ca7f7.png" height="200px" />

## Tailscale Configuration

Once deployed, unless you have [auto approval](https://tailscale.com/kb/1018/acls/#auto-approvers-for-routes-and-exit-nodes) enabled, you'll need to manually [enable the subnet routes](https://tailscale.com/kb/1019/subnets/#step-3-enable-subnet-routes-from-the-admin-console) in the tailscale console.

You'll also need to setup the nameserver. The bastion construct conveniently outputs the settings you require for Tailscale's DNS configuration:

<img src="https://user-images.githubusercontent.com/975824/177154488-3f3c1d02-35c6-432b-96fc-9dca691ea94c.png" height="250px" />

Given your configuration is correct, a direct connection to your internal resources should now be possible.
