## Notes

While reading this tutorial, you are encouraged to read before _and_ after each code block to make sure you don't execute commands with sample values. Each parameter and value that should be replaced will be mentioned and highlighted as inline code blocks (like `this`), but always be careful when copying and pasting code and commands from the internet.

Apart from that, I hope you have fun reading the tutorial.

## Pre-Requisites

You will need:

1. an AWS account
2. AWS CLI installed
3. AWS CLI properly configured

While configuring AWS CLI, you will be able to decide if you want to get the result of the operations on JSON or YAML format. Feel free to choose the format you prefer, but in this article, you will see outputs in YAML, since they are less verbose.

## Step by Step

### Registering a Domain

First, you will need to register a new domain, or use one that you have sparing. If you don't have one flying around and don't want to pay for a new one, you can use a service like https://freenom.com and register a free domain.

### Creating an AWS Hosted Zone on Route53

After that, the first thing you will have to do with AWS CLI is to created a hosted zone for your new domain under your AWS account. To do so, you can run the following command in your terminal:

```bash
aws route53 create-hosted-zone --name amnsa.org --caller-reference goforit
```

In the command above, you will be passing two arguments:

- `--name`: this will be the name of your domain
- `--caller-reference`: this will be a [random, unique value](https://docs.aws.amazon.com/Route53/latest/APIReference/API_CreateHostedZone.html#API_CreateHostedZone_RequestSyntax). 

If this command executes successfully, you will get a result like so:

```bash
ChangeInfo:
  Id: /change/C038060528Z0X8PQ4OYEK
  Status: PENDING
  SubmittedAt: '2021-06-29T22:11:40.506000+00:00'
DelegationSet:
  NameServers:
  - ns-1044.awsdns-02.org
  - ns-719.awsdns-25.net
  - ns-265.awsdns-33.com
  - ns-1983.awsdns-55.co.uk
HostedZone:
  CallerReference: goforit
  Config:
    PrivateZone: false
  Id: /hostedzone/Z080928115NXO2MVF91WL
  Name: amnsa.org.
  ResourceRecordSetCount: 2
Location: https://route53.amazonaws.com/2013-04-01/hostedzone/Z080928115NXO2MVF91WL
```

With that new hosted zone properly created, you can go to your domain registrar and configure it to point to the NS (Name Servers) pointed out by AWS on the output of the command. In the case above, there were four NS:

- `ns-1044.awsdns-02.org`
- `ns-719.awsdns-25.net`
- `ns-265.awsdns-33.com`
- `ns-1983.awsdns-55.co.uk`

### Creating an AWS S3 Bucket

After configuring Route53, the next thing you will do is to create an AWS S3 bucket where you will store static files. The following command can help you achieve that:

```bash
aws s3 mb s3://amnsa.org
```

In the command above, `mb` stands for _make bucket_ and everything after `s3://` is the name of your bucket (i.e., `amnsa.org`). This name must be a globally-unique value. That is, if someone else in the world has previously created a bucket with the name you want, you won't be able to reuse it.

Now that you got yourself a new bucket, the next thing you can do is to upload a simple HTML file to it and mark it as public to make sure everything is in place:

```bash
mkdir src
echo '<h1>Hello, World!</h1>' > ./src/index.html
aws s3 sync ./src s3://amnsa.org --acl public-read
```

There are three commands listed above:

- The first one will create a directory called `src` wherever you are (i.e., current directory)
- The second one will create a file called `index.html` inside the new directory and set its content to a "Hello, World!" message.
- The last one will push the whole contents of the `src` directory to your bucket (in this case, push `index.html` to `amnsa.org`), and mark the files as publicly accessible.

After executing these commands, you can run the following one to confirm that your file is publicly readable:

```bash
curl http://amnsa.org.s3.us-east-2.amazonaws.com/index.html
```

If your terminal prints back `<h1>Hello, World!</h1>`, then you are good to go.