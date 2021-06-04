---

title: 批量clone或pull Gitlab中所有项目

excerpt: 实用的Python脚本

date: 2021-06-04 09:17

updated: 2021-06-04 09:17

tags:
- Python

categories: Memorandum

permalink: batch-clone-or-pull-gitlab

---

~~~python
# coding=utf-8

"""
批量生成git pull 或 git clone脚本
"""

from urllib.request import urlopen
import json
import shlex
import os

gitlab_addr = 'Gitlab地址'
gitlab_token = 'Gitlab Token'
local_dir = '本地路径'

cmds = []

for index in range(10):
    url = "%s/api/v4/projects?private_token=%s&per_page=100&page=%d&order_by=name" % (
        gitlab_addr, gitlab_token, index)
    print(url)

    projects = urlopen(url)

    project_json = json.loads(projects.read().decode(encoding='UTF-8'))
    if len(project_json) == 0:
        break
    for project in project_json:
        print(project['path_with_namespace'])
        try:
            project_url = gitlab_addr + '/' + project['path_with_namespace']
            project_local_path = local_dir + '/' + project['path_with_namespace']

            if os.path.exists(project_local_path):
                cmd = shlex.split('git -C "%s" pull' % (project_local_path))
            else:
                cmd = shlex.split('git clone %s %s' % (project_url, project_local_path))

            cmds.append(' '.join(cmd))

        except Exception as e:
            print("Error on %s: %s" % (project_url, e.strerror))

print()
print()
print(*cmds, sep="\n")
print()
print()

~~~

