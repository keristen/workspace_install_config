#将下面代码放到Ubuntu根目录的.bashrc文件中，默认进入系统基础环境

```
# Ros2 和 Anaconda 隔离  *****************************************************************************
ANACONDA_HOME="/home/wzw/anaconda3"   # 这里xxx写自己的conda安装路径
 
# 备份原始PATH
if [ -z "$ORIGINAL_PATH" ]; then
    export ORIGINAL_PATH="$PATH"
    export ORIGINAL_PYTHONPATH="$PYTHONPATH"
    export ORIGINAL_LD_LIBRARY_PATH="$LD_LIBRARY_PATH"
fi
 
# 清除所有环境函数
function deactivate_all() {
    # 停用conda环境
    if command -v conda > /dev/null 2>&1 && [ -n "$CONDA_DEFAULT_ENV" ]; then
        conda deactivate 2>/dev/null || true
        
        # 清除conda相关环境变量
        unset CONDA_DEFAULT_ENV
        unset CONDA_PREFIX
        unset CONDA_SHLVL
        unset CONDA_PROMPT_MODIFIER
    fi
    
    # 清除ros2相关环境变量
    unset ROS_DISTRO
    unset ROS_VERSION
    unset ROS_PYTHON_VERSION
    unset ROS_LOCALHOST_ONLY
    unset ROS_DOMAIN_ID
    unset AMENT_PREFIX_PATH
    unset COLCON_PREFIX_PATH
    unset LD_LIBRARY_PATH
    unset PYTHONPATH
    unset ROS_OS_OVERRIDE
    
    # 从PATH中移除ros2相关路径
    export PATH=$(echo "$PATH" | sed -e 's|:/opt/ros/humble/bin||g' -e 's|/opt/ros/humble/bin:||g')
    export PATH=$(echo "$PATH" | sed -e 's|:/opt/ros/humble/lib||g' -e 's|/opt/ros/humble/lib:||g')
    
    # 从PATH中移除anaconda相关路径
    if [ -n "$ANACONDA_HOME" ]; then
        export PATH=$(echo "$PATH" | sed -e 's|:$ANACONDA_HOME/bin||g' -e 's|$ANACONDA_HOME/bin:||g')
        export PATH=$(echo "$PATH" | sed -e 's|:$ANACONDA_HOME/condabin||g' -e 's|$ANACONDA_HOME/condabin:||g')
    fi
    
    # 清除可能的遗留变量
    unset _CONDA_ROOT
    unset _CONDA_EXE
 
    # 恢复原始PATH
    export PATH="$ORIGINAL_PATH"
    export LD_LIBRARY_PATH="$ORIGINAL_LD_LIBRARY_PATH"
    # 恢复原始PYTHONPATH
    if [ -n "$ORIGINAL_PYTHONPATH" ]; then
        export PYTHONPATH="$ORIGINAL_PYTHONPATH"
    else
        unset PYTHONPATH
    fi
}
 
# 激活ros2 humble环境
function activate_ros() {
    # 首先停用所有环境
    deactivate_all
    
    # 检查ros2是否存在，这里填写自己安装的ros2路径
    if [ ! -f "/opt/ros/humble/setup.bash" ]; then
        echo "错误：未找到ros2 humble安装"
        return 1
    fi
    
    # 激活ros2
    source /opt/ros/humble/setup.bash
    
    # 设置环境指示器
    export CURRENT_ENV="ros2"
    echo "已激活ros2 humble环境"
}
 
function activate_conda() {
    # 首先停用所有环境
    deactivate_all
    
    # 检查anaconda是否存在
    if [ ! -d "$ANACONDA_HOME" ]; then
        echo "错误：未找到anaconda3安装，请检查ANACONDA_HOME变量"
        return 1
    fi
    
    # >>> conda initialize >>>
    # !! Contents within this block are managed by 'conda init' !!
    __conda_setup="$('$ANACONDA_HOME/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
    if [ $? -eq 0 ]; then
        eval "$__conda_setup"
    else
        if [ -f "$ANACONDA_HOME/etc/profile.d/conda.sh" ]; then
            . "$ANACONDA_HOME/etc/profile.d/conda.sh"
        else
            export PATH="$ANACONDA_HOME/bin:$PATH"
        fi
    fi
    unset __conda_setup
    # <<< conda initialize <<<
    
    # 激活base环境
    conda activate
    
    # 设置环境指示器
    export CURRENT_ENV="anaconda3"
    echo "已激活anaconda3环境"
}
 
function show_env() {
    if [ -n "$ROS_DISTRO" ]; then
        echo "当前环境：ROS2: $ROS_DISTRO"
    elif [ -n "$CONDA_DEFAULT_ENV" ]; then
        echo "当前环境：Anaconda3: $CONDA_DEFAULT_ENV"
    else
        echo "当前环境：基础系统环境"
    fi
}
 
# 创建别名以便快速切换
alias ros_env='activate_ros'
alias conda_env='activate_conda'
alias base_env='deactivate_all && echo "已切换到基础系统环境"'
alias current_env='show_env'
 
# 自定义bash提示符，显示当前环境
function set_prompt() {
    local last_exit_code=$?
    
    # 设置颜色
    local color_default="\[\033[0m\]"
    local color_green="\[\033[0;32m\]"
    local color_blue="\[\033[0;34m\]"
    local color_cyan="\[\033[0;36m\]"
    local color_yellow="\[\033[0;33m\]"
    local color_red="\[\033[0;31m\]"
    local color_white="\[\033[1;37m\]"
    
    # 环境指示器
    local env_indicator=""
    if [ -n "$ROS_DISTRO" ]; then
        env_indicator="${color_yellow}[ROS2:$ROS_DISTRO]${color_default}"
    elif [ -n "$CONDA_DEFAULT_ENV" ]; then
        #env_indicator="${color_cyan}[Conda:$CONDA_DEFAULT_ENV]${color_default}"
        env_indicator="${color_cyan}[$CONDA_DEFAULT_ENV]${color_default}"
    #else
        #env_indicator="${color_green}[Base]${color_default}"
    fi
    
    # 状态指示器
    #local status_indicator=""
    #if [ $last_exit_code -eq 0 ]; then
        #status_indicator="${color_green}✓${color_default}"
    #else
        #status_indicator="${color_red}✗${last_exit_code}${color_default}"
    #fi
    
    #PS1="${env_indicator} ${color_blue}\u@\h${color_default}:${color_cyan}\w${color_default}\n\$"
    #PS1="${env_indicator}${color_green}\u@\h${color_default}:${color_blue}\w${color_default}$"
        PS1="${env_indicator}${PS1_ORIGINAL}"

}
 
 
# 关键：保存系统原生PS1（必须在定义set_prompt前执行，避免被覆盖）
if [ -z "$PS1_ORIGINAL" ]; then
    export PS1_ORIGINAL="$PS1"
fi 
# 设置PROMPT_COMMAND
PROMPT_COMMAND=set_prompt
 
# 启动时显示帮助信息
echo "=========== 环境切换命令 ==========="
echo "ros_env       - 激活 ROS Humble 环境"
echo "conda_env     - 激活 Anaconda3 环境"
echo "base_env      - 返回基础系统环境"
echo "current_env   - 显示当前环境状态"
echo "=================================="


```
