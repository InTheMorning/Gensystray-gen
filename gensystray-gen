#!/bin/python

import argparse
import configparser
import os
from shutil import which


def get_settings():
    # returns a dict of configuration settings
    # from commandline arguments

    # parse commandline arguments
    parser = argparse.ArgumentParser(
        description='Generates gensystray.cfg',
        prog='gensystray-gen',
        formatter_class=argparse.ArgumentDefaultsHelpFormatter
                                     )
    parser.add_argument('NAME',
                        type=str,
                        help='a name for the config file'
                        )
    parser.add_argument('-d', '--directory',
                        type=str, metavar='<PATH>',
                        default=os.getcwd(),
                        help='a path to the directory to scan'
                        )
    parser.add_argument('-e', '--file_extension',
                        type=str, metavar='<EXTENSION>',
                        default=None,
                        help='the file extension to look for'
                        )
    parser.add_argument('-n', '--no_systemd_update',
                        action='store_true',
                        help='bypasses systemd file monitor service creation'
                        )
    parser.add_argument('-c', '--open_command',
                        type=str, metavar='<COMMAND>',
                        default='xdg-open',
                        help='command to open the files'
                        )
    args = parser.parse_args()

    # extract dict
    argsdict = vars(args)

    # this has to be hardcoded for the time being
    # because I don't know how to do this with systemd
    argsdict['gensystray_root'] = '~/.config/gensystray'

    # expand paths
    for a in ['directory', 'gensystray_root']:
        argsdict[a] = os.path.expanduser(argsdict[a])

    # convert none type to empty string for extension
    if not argsdict['file_extension']:
        argsdict['file_extension'] = ''

    return argsdict


def get_files(settings):
    # Returns a dict with names and paths for files with given extension

    directory = settings['directory']
    extension = settings['file_extension']

    # Create an empty dict
    item_pairs = {}

    # When running recursively...
    # for root, directories, files in os.walk(directory):

    # When non-recursively...
    root = next(os.walk(directory))[0]
    for filename in next(os.walk(directory))[2]:

        if not extension:
            # no extension specified, add file to dict
            filepath = os.path.join(root, filename)
            item_pairs.update({filename: filepath})
        elif filename.endswith(extension) and len(filename) > len(extension):
            # found valid extension, separate name and add to dict
            filepath = os.path.join(root, filename)
            name = filename.rsplit(extension, 1)[0]
            item_pairs.update({name: filepath})
        # file doesn't match, do nothing

    return item_pairs


def write_gensystray(settings, filedict):
    # receives dict and write to gensystray config file

    config_name = settings['NAME']
    directory = settings['directory']
    extension = settings['file_extension']
    open_command = settings['open_command']
    gensystray_root = settings['gensystray_root']

    # check if dir exists, create it if needed
    outputdir = os.path.join(gensystray_root, config_name)
    if not os.path.isdir(outputdir):
        os.mkdir(outputdir, 0o0700)

    # the output for gensystray config file
    outputfile = os.path.join(outputdir, 'gensystray.cfg')

    # the location of the icon file
    iconfile = os.path.join(outputdir, config_name + '.png')

    # use default icon if none found
    if not os.path.isfile(iconfile):
        os.popen('cp /usr/share/gensystray/gensystray.png' + ' ' + iconfile)

    # write the ini file for gensystray
    with open(outputfile, 'w') as f:
        # icon
        f.write('@' + iconfile + '\n\n')

        # tooltip
        f.write("'" + config_name + "'" + '\n\n\n')

        # all items
        for filename, filepath in filedict.items():
            f.write('[%s]\n%s %s\n\n' % (filename, open_command, filepath))

        # [New file] section
        # zenity command string that gets evaluated
        zenity_cmd = ("TITLE=$(zenity --entry --text='New file name:' "
                      "--title='Add new file' --entry-text 'Untitled') "
                      "|| exit 1 && ")

        # construct the newfile command string
        newfile_command = (zenity_cmd + open_command + ' '
                           + directory + '$TITLE' + extension)
        # surround it with separators and write it
        f.write('[%s]\n%s\n\n' % ('-', '-'))
        f.write('[%s]\n%s\n\n' % ('New File', newfile_command))
        f.write('[%s]\n%s\n\n' % ('-', '-'))
    print('wrote %s' % outputfile)


def write_systemd(settings, exec_path):
    # Make systemd .path and .service files

    config_name = settings['NAME']
    directory = settings['directory']
    extension = settings['file_extension']
    open_command = settings['open_command']
    servicename = 'gensystray-refresh@'
    pathfile_name = servicename + config_name + '.path'
    unitfile_name = servicename + config_name + '.service'
    unitfile_string = '%s%si.service' % (servicename, '%')
    execstart_string = (
                        '%s %s %s %s %s %s %s %s %s' %
                        (exec_path, config_name, '-n', '-c', open_command,
                         '-d', directory, '-e', extension)
                         )
    outputdir = os.path.expanduser('~/.config/systemd/user/')
    output_pathfile = os.path.join(outputdir, pathfile_name)
    output_unitfile = os.path.join(outputdir, unitfile_name)

    # create the .path and .service files
    pathfile = configparser.ConfigParser(interpolation=None)
    pathfile.optionxform = str
    pathfile['Unit'] = {'Description': 'Monitor a folder file for changes'}
    pathfile['Path'] = {'PathModified': directory,
                        'Unit': unitfile_string}
    pathfile['Install'] = {'WantedBy': 'multi.user.target'}

    unitfile = configparser.ConfigParser(interpolation=None)
    unitfile.optionxform = str
    unitfile['Unit'] = {'Description':
                        'Refresh gensystray config after file changes'}
    unitfile['Service'] = {'Type': 'oneshot',
                           'ExecStart': execstart_string}
    unitfile['Install'] = {'WantedBy': 'multi.user.target'}

    # write the files in ini format
    with open(output_pathfile, 'w') as f:
        pathfile.write(f)
    print('wrote %s' % output_pathfile)
    with open(output_unitfile, 'w') as f:
        unitfile.write(f)
    print('wrote %s' % output_unitfile)

    # reload and enable main .service
    os.system('systemctl --user daemon-reload')
    os.system('systemctl --user enable gensystray@%s.service' %
              config_name)
    os.system('systemctl --user start gensystray@%s.service' %
              config_name)


if __name__ == "__main__":
    settings = get_settings()
    filedict = get_files(settings)

    write_gensystray(settings, filedict)
    if not settings['no_systemd_update']:
        exec_path = which('gensystray-gen')
        if exec_path:
            write_systemd(settings, exec_path)
        else:
            print("gensystray-gen exectutable not found in path.")
            raise SystemExit("Error: Can not create systemd files, Exiting")
