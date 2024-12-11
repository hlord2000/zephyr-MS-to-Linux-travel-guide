# A Zephyr Developer's Windows-to-Linux Travel Guide

If you are accustomed to using Windows for Zephyr development and Linux-curious you probably have a few questions.
What is a virtual environment? Why do I get this strange warning when I try to install west?

```
error: externally-managed-environment

× This environment is externally managed
╰─> To install Python packages system-wide, try apt install
    python3-xyz, where xyz is the package you are trying to
    install.
    
    If you wish to install a non-Debian-packaged Python package,
    create a virtual environment using python3 -m venv path/to/venv.
    Then use path/to/venv/bin/python and path/to/venv/bin/pip. Make
    sure you have python3-full installed.
    
    If you wish to install a non-Debian packaged Python application,
    it may be easiest to use pipx install xyz, which will manage a
    virtual environment for you. Make sure you have pipx installed.
    
    See /usr/share/doc/python3.12/README.venv for more information.
```

What are Nordic's VS code extensions *really* doing in the background? How do I get started!?

Why even bother?

Questions abound. But the benefits of Linux for Zephyr development are numerous:
- Slash your build times by half. Some report a 90% improvement in compile time.
- Use the same development environment Zephyr contributors use.
- Easy extensibility and customization.
- Fewer cryptic toolchain issues.

What follows is a step-by-step guide and explanation to the process of making your Linux development environment comfy AND cozy.

# On Zephyr's build system for build systems for build systems
West is Zephyr's meta-build system tool. It is an open-source and extensible Python utility:
https://github.com/zephyrproject-rtos/west

It greatly simplifies the process of managing (and making reproducible!) your development environment, especially with projects that use many different repositories. There is an inordinate amount of CMake glue supplied by the Zephyr Project to make this process easy with libraries as disparate as:
- LVGL
- LittleFS
- TrustedFirmware-M
- Matter
- zscilib (a Zephyr scientific utility library)
- Vendor HALs (Nordic, ST, Infineon, etc.)
... The list goes on. To see what is available check out the west.yml file in Zephyr:
https://github.com/zephyrproject-rtos/zephyr/blob/main/west.yml

Among other things, west handles installation, updating, and use of these libraries. The west.yml file above can be modified to include only the repositories you need for your project. For an excellent example of how this can be done, Zephyr supplies a project with some of these features:

https://github.com/zephyrproject-rtos/example-application

West is a very complex tool. This complexity and the wide variety of dependencies it pulls in necessitates the use of Python "virtual environments." Consider a virtual environment to be a fresh installation of Python with the **exact** dependencies that your project needs. Python's most commonly used package manager is called "pip," whose usage you will see shortly. The warning above is downstream of the Python community's strong recommendation to use virtual environments. It also helps make your builds repeatable.

Note that you will often see the use of pip's "requirements.txt" files. This is effectively the same as manually typing out "pip install <foo>" for each dependency.

# Step 0: Install Zephyr/nRF Connect SDK

Follow the instructions for Zephyr here: https://docs.zephyrproject.org/latest/develop/getting_started/index.html

Follow the instructions for nRF Connect SDK here: https://docs.nordicsemi.com/bundle/ncs-latest/page/nrf/installation/install_ncs.html

Be sure to note the location of your virtual environment. I use the /opt directory for my SDK and toolchain installations. 

```
├── ncs
│   ├── ncs-main
│   ├── ncs-v2.7
│   └── ncs-v2.8
├── zephyr-main
│   ├── bootloader
│   ├── modules
│   ├── tools
│   ├── venv
│   └── zephyr
└── zephyr-sdk-0.17.0
...
```

# Step 1: Activate virtual environment, source zephyr-env.sh, and enable autocomplete

There are a few different ways to configure your Zephyr build environment. Please thoroughly read the documentation on the different configurations here:

https://docs.zephyrproject.org/latest/develop/west/workspaces.html

The following script is what I use to activate my virtual environments.

```zsh
function actvenv_ncs() {
    local ncs_dir="/opt/ncs"
    local version="$1"
    
    if [[ -z "$version" ]]; then
        # If no version is provided, find the highest numbered one
        version=$(ls -d ${ncs_dir}/ncs-* | sort -V | tail -n 1 | xargs basename)
    else
        # If a version is provided, ensure it's prefixed with "ncs-v"
        version="ncs-${version#ncs-}"
    fi
    
    local full_path="${ncs_dir}/${version}"
    
    if [[ ! -d "$full_path" ]]; then
        echo "Error: Directory $full_path does not exist."
        return 1
    fi
    
    source "${full_path}/venv/bin/activate" && source "${full_path}/zephyr/zephyr-env.sh" && source <(west completion zsh)
    echo "Activated environment for ${version}"
}

# Add autocompletion
_actvenv_ncs() {
    local ncs_dir="/opt/ncs"
    _values 'versions' ${ncs_dir}/ncs-*(/:t)
}
compdef _actvenv_ncs actvenv_ncs
```

In short, it searches for matching virtual environments in the /opt/ncs directory, activates the virtual environment, sources the zephyr-environment.sh script (which tells west which SDK location to use), and enables west autocompletion.

West autocomplete greatly assists in building! Hit tab with just a substring of your board target, among other things.

