# Ultimate Shim 

This repository contains a bash script that serves as a "shim" for running various application commands with setup and environment configurations.

## Description

The script works as follows:

1. First, it checks for a `.app_name.env` file for the application in the current directory and sources it if found.
2. Next, it checks for a `.app_name.setup` file or directory for the application in the current directory. If a file is found, it's executed. If a directory is found, all files within that directory are executed.
3. If any of the `setup` scripts return an error code, the shim stops executing and doesn't run the actual application
4. Finally, it runs the actual application command

The script also provides an option to temporarily skip the shim by setting an environment variable `NO_{APP_NAME}_SHIM` to a truthy value. 

## Naming and Softlinking the Shim

In order to successfully use the shim script, you need to either name the shim script the same name as the application being shimmed, or create a softlink with the application's name that points to the shim script.

### Renaming the Shim

Rename the shim to have the same name as the application you're intending to shim. For instance, if you're planning to shim an application called `my_app`, rename your shim script to `my_app`.

```bash
mv shim_script.sh my_app
```

### Creating a Softlink

If you don't want to rename your shim, you can create a softlink with the application's name that points to the shim script. Here's how you'd do it:

```bash
ln -s /path/to/shim_script.sh /path/to/my_app
```

Make sure to replace `/path/to/shim_script.sh` with the actual path to your shim script, and `/path/to/my_app` with where you want the softlink to reside.

### Path Considerations

Whether you choose to rename the shim or create a softlink, you need to make sure the resulting script or link is placed in a directory at the beginning of your `PATH`. This is to ensure that when you run the application command, the shim gets called instead of the actual application being shimmed.

You can achieve this by adding the directory to your `PATH` in your shell profile (`~/.bashrc`, `~/.bash_profile`, `~/.zshrc`, etc.):

```bash
export PATH="/path/to/shim_directory:$PATH"
```

Again, replace `/path/to/shim_directory` with the actual path to the directory holding your shim or softlink.

**IMPORTANT:** These changes will take effect in new shell sessions. To apply them in the current shell, use the `source` command:

```bash
source ~/.bashrc # or ~/.bash_profile, ~/.zshrc, etc.
```

Now, when you call the application command, it will run the shim script instead of the actual application. The shim script will then set up the environment, perform any necessary setup, and finally run the actual application.

## Environment Configuration

You can create a `.env` file for your application with any environment variables your app needs. This file should be in the same directory from which you run the shim script. 

Example `.env` file:

```bash
# .app_name.env
export VARIABLE_1=value1
export VARIABLE_2=value2
```

## Setup Configuration

If your application requires any setup steps before it can be run, you can define them in a `.app_name.setup` file or directory for your application. If a directory, all files within the directory will be executed. This should also be in the same directory you run the script from.

Example `.app_name.setup` file:

```bash
# .app_name.setup
echo "Performing setup..."
# your setup commands here
```

## Example: Creating a Shim for Rails

In this section, we will walk through the process of creating a shim for Rails, along with a `.setup` script that checks if the PostgreSQL port is running and a `.env` file that sets the `PORT` variable to 9000.

### 1. Preparing the Shim

First, clone or download the `ultimate_shim` bash script from this repository to your local machine.

### 2. Naming/Linking the Shim

You can rename the `ultimate_shim` to `rails`:

```bash
cp ultimate_shim $HOME/mybins/rails
```

Or create a softlink named `rails` pointing to `ultimate_shim`:

```bash
ln -s /path/to/ultimate_shim $HOME/mybins/rails
```

Replace `/path/to/ultimate_shim` with the actual path where `ultimate_shim` resides.

### 3. Setting up the Path

Next, add `$HOME/mybins` directory to the beginning of your `PATH` in your shell profile (`~/.bashrc`, `~/.bash_profile`, `~/.zshrc`, etc.):

```bash
echo 'export PATH="$HOME/mybins:$PATH"' >> ~/.bashrc
```

Source your profile to apply the changes in the current shell:

```bash
source ~/.bashrc # or ~/.bash_profile, ~/.zshrc, etc.
```

### 4. Creating .env file

Create a `.rails.env` file in the directory where you will run the Rails command, and set the `PORT` variable to 9000:

```bash
echo 'export PORT=9000' > .rails.env
```

### 5. Creating .setup file

Create a `.rails.setup` file that checks if the PostgreSQL port is running:

```bash
echo 'lsof -i:5432 > /dev/null || { echo "PostgreSQL is not running"; exit 1; }' > .rails.setup
```

This command checks if any process is listening on port 5432 (default port for PostgreSQL). If no process is found, it prints a message and exits with an error code.

### 6. Running Rails with the Shim

Now, when you call `rails` from the directory containing `.rails.env` and `.rails.setup`, it will run the shim instead of the actual Rails command. The shim will source the `.rails.env`, run `.rails.setup`, and then execute the actual Rails command.

For example, to start the Rails server:

```bash
rails server
```

This will start the Rails server with the `PORT` environment variable set to 9000, but only if PostgreSQL is running. If PostgreSQL is not running, the server will not start, and you'll see a "PostgreSQL is not running" message.

## Errors

In case of any error during the setup phase (if the `.setup` file or any file within the `.setup` directory returns a non-zero exit code), the script will exit without running the actual app command if the first argument is 'c' or 's'. However, for any other argument, the script will still run the actual app command, despite the setup error.

This c or s exception will go away on the next version

## Notes

This script assumes the original application executable is in the `PATH`. If it finds the shim instead of the original executable, or if it doesn't find the original executable at all, it will exit with an error.

If you have any questions or issues, feel free to open an issue in this repository.

