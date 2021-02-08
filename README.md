# Learning CloudFormation AWS

CloudFormation Templates (CFT) let you build stacks in AWS and create resources.
The `sever-UI-ec2s-cloudformation.yaml` creates a react, nginx ec2 (UI) in a given public subnet and a node express ec2 (Backend) in a given private subnet which talks to a mysql database pre made. They both also have reference to security groups already made.

Prior to this, a VPC, IGW, Route tables, RDS (mysql), Security groups, NAT Gateway and a public & private subnet had been created on AWS with the id references in the CFT.

## Notes

# Speaking to ports
Remember if a node server is running on port 8080, set the inbound rules of that EC2 security group to allow custom TCP connections to port 8080 of 0.0.0.0/0. Also the same for outbound rules of EC2 security groups connecting to it.

# EC2 in public subnet to speak to EC2 in private
Nginx has been used to proxy requests to the EC2 in the private subnet. Without this, the requests will be made from the browser and not the UI. The browser/internet cannot speak to the EC2 in the private subnet but the EC2 in the public can because it is in the same network.

Express can be set up to do the same.

# Keep the server running on reboot
Running node index.js works until the ec2 restarts or you exit the terminal. Use pm2 to keep it running.

Another way is to keep your node app running with `systemd` or docker containers.

```
# install pm2 to keep the server running on reboot
npm i pm2 -g

# print the following to pm2.config.js file (ubuntu is the user on the OS)
cat << EOF >> /home/ubuntu/pm2.config.js
module.exports = {
    apps : [
        {
            name: "todo",
            script: "/home/ubuntu/todo-server/index.js",
            watch: true,
            env: {
                "PORT": 8080,
                "NODE_ENV": "development",
                "DB_HOST": "${DBHost}",
                "DB_USER": "${DBUser}",
                "DB_PASS": "${DBPass}"
            },
            env_production: {
                "PORT": 80,
                "NODE_ENV": "production",
            }
        }
    ]
}
EOF

pm2 start /home/ubuntu/pm2.config.js --env development
pm2 list

# start on reboot
pm2 startup
pm2 save
```

### Run script in a different user

When user data is run on an EC2, it runs as root. When you ssh into an ec2 you are a different user (ubuntu, ec2-user...) depending on the OS.

Include the following in user data and it will run the script in the given user (ubuntu because of the AMI).
```
# Everything between the EOF goes into the file given (subscript.sh)
cat > /tmp/subscript.sh << EOF
#!/bin/bash
whoami >
EOF

chown ubuntu:ubuntu /tmp/subscript.sh && chmod a+x /tmp/subscript.sh
sleep 1; su - ubuntu -c "/tmp/subscript.sh"
```