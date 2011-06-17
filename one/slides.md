!SLIDE

# The Butler and the Chef

!SLIDE bullets
# Jason Cook                                                                                        
* jasonc@simpleideas.org                                                                            
* github.com/macros                                                                                 
* twitter.com/macros                                                                                
* freenode #macros  

!SLIDE center
# ![Wikia](wikia.png)                                                                                
(Hiring for ops and dev)

!SLIDE center
# NewCo
(launching soon!)

!SLIDE 
# We have devs
## (All with ops backgrounds)

!SLIDE
# Need a way to deploy

!SLIDE 
# Has to be simple
## Everyone should be able to push

!SLIDE 
# Has to be reliable
## Both dev and ops need to trust it

!SLIDE 
# Has to transparent
## No guessing about who deployed
## Accurate record of what was deployed

!SLIDE
# So the how

!SLIDE center
# ![BORK](swedish_chef_bork-sleeper-cell.jpg) #

!SLIDE
# Chef
## Great for managing system state

!SLIDE 
# The application is part of that state

!SLIDE
# Application state
    @@@json
    {        
      "id": "sierra-staging",
      "port": "6001",
      "version": "2011-03-25_07-51-39"
    }

!SLIDE
# Deploy logic in cookbooks

!SLIDE
# App dependencies managed in chef
## Aim to use os packages everywhere possible

!SLIDE
# Data bags are epheremal
## Great for defining a state
## Not a history of state
## Hand editing data bags sucks
### And too easy to screw up

!SLIDE
# But deploy is only part of the process

!SLIDE
# Jenkins
## aka Hudson
### ( Bad Oracle )

!SLIDE
# Nice step wise scripting environment
## Oriented towards building software

!SLIDE
# Continuous Integration Server
## Runs your build scripts
## Runs your tests
## Trigger events
## Archive the artifacts

!SLIDE
# Nice plugin ecosystem
## Including ruby support

!SLIDE
# Our deploy workflow

!SLIDE
# Developer merges changes
## Production and Staging branches

!SLIDE
# Developer starts the build
## Via IRC or Web
## Notifies IRC

!SLIDE
# Jenkins builds the code
## Not always needed

!SLIDE
# Jenkins runs the tests
    @@@perl
    prove -I ./lib -v --harness=TAP::Harness::JUnit

!SLIDE
# Archives a tar ball of the app

!SLIDE
# Updates the app cookbook
    @@@ruby
    require 'json'
    app = JSON.parse(`knife data bag -c
    /var/lib/jenkins/.chef/knife.rb show apps sierra-staging`)
    app["version"] = ENV['BUILD_ID']
    File.open("sierra-staging.json", 'w') {|f| f.write(app.to_json)}

!SLIDE
# Runs chef-client on the app nodes
    knife data bag -c /var/lib/jenkins/.chef/knife.rb \
    from file apps sierra-staging.json
    knife ssh -i /var/lib/jenkins/.ssh/jenkins-chef \
    -x root -c /var/lib/jenkins/.chef/knife.rb \ 
    "role:app" "chef-client"

!SLIDE
# Error handling during deploy is chef exception handlers
## Not sure ideal, but works for now

!SLIDE    
# Rollback currently manual
## Plan to move to a knife plugin

!SLIDE
# Jenkins not just for dev

!SLIDE
# Ops manages artifacts in the system
## Ubuntu packages

!SLIDE
# Ops Workflow

!SLIDE
# Add a build
## Git or external source

!SLIDE
# Generate the .dsc
## Only needed for internal trees
    dpkg-source -b varnish-cache
    
!SLIDE
# Run a pbuilder to build
    sudo DIST=lucid /usr/sbin/pbuilder --build  \
    --buildresult ${WORKSPACE}/build varnish*.dsc 

!SLIDE
# Add to our internal repo
    cp build/*.deb /srv/repo/Varnish/
    cd /srv/repo/Varnish
    dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz

!SLIDE
# Future Work
## knife plugin
## statsd notify on push
## Better exception notification
## State machine driven push


!SLIDE
# Thanks!
### jasonc@simpleideas.org