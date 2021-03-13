1. follow this turorial to create ssh key on the server https://docs.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
2. follow this  tutorial to add the key to github account  https://docs.github.com/en/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account


3. create a webhook url in urls.py `    url(r'^5168d7d4-dbb9-4d61-8ee9-2e596afaee3a/', GitHubDeployView.as_view(), name='deploy'),`

4. create a webhook view 
``` 
@method_decorator(csrf_exempt, name='dispatch')
class GitHubDeployView(View):
    def post(self, request):
        if self.validate_signature(request):
            run_git = subprocess.Popen(['make'], cwd='/home/ubuntu/website/build', shell=False, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
            # stdout, stderr = run_git.communicate()
            # logger.info(f"{stdout}, {stderr}")
            return HttpResponse(status=200)
        return HttpResponse(status=401)
    
    def validate_signature(self, request):
        key = settings.GH_SECRET.encode()
        expected_signature = hmac.new(key=key, msg=request.body, digestmod=hashlib.sha256).hexdigest()
        incoming_signature = request.headers.get('X-Hub-Signature-256').split('sha256=')[-1].strip()
        if not hmac.compare_digest(incoming_signature, expected_signature):
            return False
        return True
```
        
5. Setup webhook with url on the github repo settings you want to ci-cd
6. create a makefile 
```
#!/bin/bash
SHELL=/bin/bash

deploy:
	eval `ssh-agent -s` && ssh-add /home/ubuntu/.ssh/id_ed25519 && ssh-add -l && git pull origin master && rm -f /home/ubuntu/website/static/js/main* && source /home/ubuntu/website/webenv/bin/activate && python /home/ubuntu/website/manage.py collectstatic --no-input && sudo service gunicorn restart
 ```
 
 7. makesure to change the folder names according both in view and makefile.
 8. set up git remote ``` git remote rm origin && git remote add origin 'git@github.com:hekafy/SEP_build_react.git' ```

This should be good enough for the CI-CD
