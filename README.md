Forensic-Python-Script
======================

Forensic Analysis Python Script. Will accept any disk image and analyze it accordingly. This is a project I had done for school over the summer. The script will determine whether the disk is a full disk image or a single partition and the user will be prompted with forensic analysis options such as printing a listing of all files (deleted or not), displaying the disk's MD5 sum, determining the file system type, and giving the total number of files and directories on disk


The script relies on The Sleuth Kit (TSK) forensic suite and was run and tested on a system with TSK already installed


#!/usr/bin/python
import sys, os, re, subprocess
from subprocess import call

#Function Used to Calculate MD5 Hash
def md5calc():
	m = subprocess.Popen(["md5sum", sys.argv[1]], stdout=subprocess.PIPE)
	sum, err = m.communicate()
	print "The MD5 Sum of the disk image is:", sum

#Function which reads the disk image and determines if it is a full disk or single partition
def read_image():
	p = subprocess.Popen(["file", sys.argv[1]], stdout=subprocess.PIPE)
	out, err = p.communicate()
	if "partition" in out:
		print
		print "This is a full disk image. Below are the STARTING SECTORS of each partition:"
		print
		p1 = subprocess.Popen(["fdisk", "-luc", sys.argv[1]], stdout=subprocess.PIPE)
		p2 = subprocess.Popen(["grep", "-Ev", "Extended"], stdin=p1.stdout, stdout=subprocess.PIPE)
                p3 = subprocess.Popen(["sed", "1,9d"], stdin=p2.stdout, stdout=subprocess.PIPE)
		p4 = subprocess.Popen(["awk", "{print $2}"], stdin=p3.stdout, stdout=subprocess.PIPE)
                output = p4.communicate()[0]
                print
		print output
		part_selection = raw_input("Enter the STARTING SECTOR of the partition you wish to analyze: ")
		
		p = subprocess.Popen(["fsstat", "-o", part_selection, sys.argv[1]], stdout=subprocess.PIPE)
                stat, err = p.communicate()
		
		#Check for error on FSSTAT command
		while (p.returncode) != 0:
			part_selection = raw_input("This is an invalid number. Choose again: ")
			p = subprocess.Popen(["fsstat", "-o", part_selection, sys.argv[1]], stdout=subprocess.PIPE)
                	stat, err = p.communicate()
                        
              
		################ THIS SECTION DISPLAYS THE MENU SCREEN FOR THE SPECIFIED PARTITION #######################
		menu_choice = []

        	while menu_choice != "9":
                	print "--------------------"
                	print "1. MD5 Sum"
                	print "2. File System Type"
                	print "3. Number of Files and Directories on Disk"
                	print "4. Number of Deleted Files"
                	print "5. List All Files on Disk"
			print "6. List All Deleted Files on Disk"
			print "9. Quit"
                	print
                	menu_choice = raw_input("Pick an item from the menu: ")
                	if menu_choice == "1":
                        	print
                        	print "Calculating MD5 Sum. This may take a few minutes."
                        	print
                        	md5calc()

                	elif menu_choice == "2":
                        	p1 = subprocess.Popen(["fsstat", "-o", part_selection, "-t", sys.argv[1]], stdout=subprocess.PIPE)
                        	#p2 = subprocess.Popen(["grep", "-a", "File System Type:"], stdin=p1.stdout, stdout=subprocess.PIPE)
                        	output = p1.communicate()[0]
                        	print
                	        print output
		
                	elif menu_choice == "3":
                	        c1 = subprocess.Popen(["fls", "-o", part_selection, "-r" , sys.argv[1]], stdout=subprocess.PIPE)
                	        c2 = subprocess.Popen(["wc", "-l"], stdin=c1.stdout, stdout=subprocess.PIPE)
                	        output = c2.communicate()[0]
                	        print
                	        print "There are", output, "files and directories"
	
        	        elif menu_choice == "4":
                	        p1 = subprocess.Popen(["fls", "-o", part_selection, "-d", sys.argv[1]], stdout=subprocess.PIPE)
                	        p2 = subprocess.Popen(["wc", "-l"], stdin=p1.stdout, stdout=subprocess.PIPE)
                	        output = p2.communicate()[0]
                	        print
                	        print "There are", output, "deleted files"
	
			elif menu_choice == "5":
                                p1 = subprocess.Popen(["fls", "-o", part_selection, "-r", sys.argv[1]], stdout=subprocess.PIPE)
                                output = p1.communicate()[0]
                                print output
				answer = raw_input("Would you like to save this list to a file (y / n): ")
				while answer != "y" and answer != "n":
					answer = raw_input("This is not a valid choice. Please answer y / n : ")

				if answer == "n":
					print "OK"
				elif answer == "y":
					new_file = raw_input("Enter the file name you wish to save the list to: ")
					while os.path.exists(new_file):
						print "This file name already exists. Cannot save list here."
						new_file = raw_input("Please enter a new file name: ")
					else:
						out_file = open(new_file, "w")
						out_file.write(output)
						out_file.close()


			elif menu_choice == "6":
                                p1 = subprocess.Popen(["fls", "-o", part_selection, "-r", "-d", sys.argv[1]], stdout=subprocess.PIPE)
                                output = p1.communicate()[0]
                                print output
                                answer = raw_input("Would you like to save this list to a file (y / n): ")
                                while answer != "y" and answer != "n":
                                        answer = raw_input("This is not a valid choice. Please answer y / n : ")

                                if answer == "n":
                                        print "OK"
                                elif answer == "y":
                                        new_file = raw_input("Enter the file name you wish to save the list to: ")
                                        while os.path.exists(new_file):
                                                print "This file name already exists. Cannot save list here."
                                                new_file = raw_input("Please enter a new file name: ")
                                        else:
                                                out_file = open(new_file, "w")
                                                out_file.write(output)
                                                out_file.close()



        	        elif menu_choice == "9":
        	                print "Goodbye"
        	        else:
                	        print
                	        print "Not a valid selection. Please choose again."
                        	print


		              	

		print
	# If "partition" is not found, this means the disk is a single partition
	# Call the next function, menu
	else:
		menu()

#Function used for single partition images
def menu():
	menu_choice = []
	
	while menu_choice != "9":
		print "--------------------"
		print "1. MD5 Sum"
		print "2. File System Type"
		print "3. Number of Files and Directories on Disk"
		print "4. Number of Deleted Files"
		print "5. List All Files on Disk"
		print "6. List All Deleted Files on Disk"
		print "9. Quit"
		print
		menu_choice = raw_input("Pick an item from the menu: ")
		if menu_choice == "1":
			print
			print "Calculating MD5 Sum. This may take a few minutes."
			print
			md5calc()
			
		elif menu_choice == "2":
			p1 = subprocess.Popen(["fsstat", "-t", sys.argv[1]], stdout=subprocess.PIPE)
                        output = p1.communicate()[0]
                        print
                        print output

		elif menu_choice == "3":
			c1 = subprocess.Popen(["fls", "-r" , sys.argv[1]], stdout=subprocess.PIPE)
                        c2 = subprocess.Popen(["wc", "-l"], stdin=c1.stdout, stdout=subprocess.PIPE)
                        output = c2.communicate()[0]
                        print
                        print "There are", output, "files and directories"
		
		elif menu_choice == "4":
			p1 = subprocess.Popen(["fls", "-d", sys.argv[1]], stdout=subprocess.PIPE)
			p2 = subprocess.Popen(["wc", "-l"], stdin=p1.stdout, stdout=subprocess.PIPE)
			output = p2.communicate()[0]
			print
			print "There are", output, "deleted files"

		elif menu_choice == "5":
			p1 = subprocess.Popen(["fls", "-r", sys.argv[1]], stdout=subprocess.PIPE)
			output = p1.communicate()[0]
                        print output
                        answer = raw_input("Would you like to save this list to a file (y / n): ")
                        while answer != "y" and answer != "n":
				answer = raw_input("This is not a valid choice. Please answer y / n : ")
			if answer == "n":
				print "OK"
			elif answer == "y":
				new_file = raw_input("Enter the file name you wish to save the list to: ")
				while os.path.exists(new_file):
					print "This file name already exists. Cannot save list here."
                                        new_file = raw_input("Please enter a new file name: ")
				else:
					out_file = open(new_file, "w")
                                	out_file.write(output)
                                	out_file.close()

		elif menu_choice == "6":
                        p1 = subprocess.Popen(["fls", "-r", "-d", sys.argv[1]], stdout=subprocess.PIPE)
                        output = p1.communicate()[0]
                        print output
                        answer = raw_input("Would you like to save this list to a file (y / n): ")
                        while answer != "y" and answer != "n":
                                answer = raw_input("This is not a valid choice. Please answer y / n : ")
                        if answer == "n":
                                print "OK"
                        elif answer == "y":
                                new_file = raw_input("Enter the file name you wish to save the list to: ")
                                while os.path.exists(new_file):
                                        print "This file name already exists. Cannot save list here."
                                        new_file = raw_input("Please enter a new file name: ")
                                else:
                                        out_file = open(new_file, "w")
                                        out_file.write(output)
                                        out_file.close()

		elif menu_choice == "9":
			print "Goodbye"
		else:
			print
			print "Not a valid selection. Please choose again."
			print
	

######### main #######

if len(sys.argv) > 2 or len(sys.argv) < 2:
	print "Usage: myscript.py imagefile"
else: 
	if os.path.exists(sys.argv[1]):	
		p = subprocess.Popen(["file", sys.argv[1]], stdout=subprocess.PIPE)
		out, err = p.communicate()
		if "boot sector" in out:	
			read_image()
		else:
			print "This is not a valid disk image"
			
	else:
		print "File", sys.argv[1], "could not be found"
