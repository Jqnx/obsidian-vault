Tide Colors
[dotfiles/fish/conf.d/tide.fish at master · Jqnx/dotfiles · GitHub](https://github.com/Jqnx/dotfiles/blob/master/fish/conf.d/tide.fish)

```bash
#----------------------
# character
#----------------------

set -g tide_character_color ca9ee6
set -g tide_character_color_failure e78284

#----------------------
# cmd_duration
#----------------------

set -g tide_cmd_duration_color 81c8be

#----------------------
# docker
#----------------------

set -g tide_docker_color 85c1dc

#----------------------
# git
#----------------------

set -g tide_git_color_branch 51576d
set -g tide_git_color_conflicted e78284
set -g tide_git_color_dirty ea999c
set -g tide_git_color_operation e78284
set -g tide_git_color_staged ea999c
set -g tide_git_color_stash ca9ee6
set -g tide_git_color_untracked 8caaee
set -g tide_git_color_upstream ca9ee6

#----------------------
# kubectl
#----------------------

set -g tide_kubectl_color 8caaee
_tide_find_and_remove kubectl tide_right_prompt_items

#----------------------
# node
#----------------------

set -g tide_node_color 81c8be

#----------------------
# pwd
#----------------------

set -g tide_pwd_color_anchors babbf1
set -g tide_pwd_color_dirs babbf1
set -g tide_pwd_color_truncated_dirs 626880

#----------------------
# status
#----------------------

set -g tide_status_color a6d189
set -g tide_status_color_failure e78284

#----------------------
# terraform
#----------------------

set -g tide_terraform_color ca9ee6

#----------------------
# time
#----------------------

set -g tide_time_color babbf1
```
