This short documentation describes steps necessary to compile and run the human pose estimation model presented in the paper:

Leonid Pishchulin, Micha Andriluka, Peter Gehler and Bernt Schiele
Strong Appearance and Expressive Spatial Models for Human Pose Estimation
IEEE International Conference on Computer Vision (ICCV'13), Sydney, Australia, December 2013

This code was developed under Linux (Debian wheezy, 64 bit) and was tested only in this environment.
If you have any questions, send an email to leonid@mpi-inf.mpg.de with a topic "partapp code".
It may also be possible that we run our model on your data.

1. Required Libraries

   The following libraries are required to compile and run the code:
   
   - Qt (tested with version 4.8.2)
   - Boost (tested with version 1.34), http://www.boost.org/
   - Matlab (tested with Matlab 2008b)
   - Goolgle's Protocol Buffers (tested with version 2.0.1rc1), http://code.google.com/p/protobuf/

2. Compiling the code

   - Switch to the top level directory of the source code (the one with the script "run_partapp.sh").
   - Issue commands:
   
     ln -s ./external_include/matlab-2008b include_mat	
     ln -s ./external_lib/matlab-2008b-glnxa64-nozlib lib_mat		
     ln -s ./external_include include_pb
     ln -s ./external_lib lib_pb
     
   - add the full path of folder "include_mat" to the LD_LIBRARY_PATH environment variable
   - point to your matlab runtime environment by editing the file src/libs/libPrediction/matlab_runtime.h
   - issue commands "qmake-qt4 -recursive; make; ./compileMatlab.sh" in the "src/libs" directory
   - issue commands "qmake-qt4 -recursive; make" in the "src/apps" directory;

3. Test the compiled code  

   - Issue the following commands in the code_test subdirectory:
   
	unzip code_test.zip
      	../run_partapp.sh --expopt ./expopt/exp-code-test-local-app-model.txt --head_detect_dpm --part_detect_dpm --find_obj
        ../run_partapp.sh --expopt ./expopt/exp-code-test-poselets.txt --save_resp_test
   	../run_partapp.sh --expopt ./expopt/exp-code-test-full-model.txt --find_obj --eval_segments --vis_segments

   This will run local appearance model, compute poselet responses and finally run full model to estimate body parts 
   on a provided image and visualize the results. 
   Compare the image in the ./log_dir/exp-code-test-full-model/part_marginals/seg_eval_images with the image
   in ./images_result

4. Running pose estimation experiments
   
   Our model requires that training and testing images contain single persons being roughly 200px high.
   
   Download the experiments package from our homepage: https://www.d2.mpi-inf.mpg.de/poselet-conditioned-ps
   Unpack the package in the separate directory <EXP_DIR>. Here is a short description of the contents:
 
   ./images/LSP - upscaled images from the LSP dataset, so that every person is roughly 200px high
   ./expopt - configuration files which control diverse parameters of the system
   
   ./log_dir/exp-lsp-local-app-model/class - pretrained AdaBoost models (classifiers)
   ./log_dir/exp-lsp-local-app-model/dpm_model - pretrained DPM models
   ./log_dir/exp-lsp-local-app-model/spatial - pretrained generic spatial model
   ./log_dir/exp-lsp-local-app-model/test_dpm_unary/torso - responses of the DPM torso detector

   ./log_dir/exp-lsp-train-torso/part_marginals - torso detections on the train set
   
   ./log_dir/exp-lsp-poselets/class - pretrained AdaBoost poselet models
   ./log_dir/exp-lsp-poselets/resp_train - poselet responses on train images
   ./log_dir/exp-lsp-poselets/pred_data - pretrained LDA classifiers to predict poselet conditioned parameters
   
   ./log_dir/exp-lsp-full-model/spatial - precomputed poselet conditioned spatial model
   
   1) Test

   Assuming that <PARTAPP_DIR> is the directory where you have unpacked and compiled the source code, you can run 
   the system on a single image by issuing the commands:
       	      
   	      <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-local-app-model.txt --head_detect_dpm --part_detect_dpm --find_obj --first <IMGIDX> --numimgs 1
	      <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-poselets.txt --save_resp_test --first <IMGIDX> --numimgs 1
	      <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-full-model.txt --find_obj --first <IMGIDX> --numimgs 1
      
   where <IMGIDX> is index of the image (if "first" and "numimgs" parameters are omitted the whole dataset will be processed).	 

   In order to evaluate the number of correctly detected parts and visualize the results run:
        
	      <PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-full-model.txt --eval_segments --vis_segments --first <IMGIDX> --numimgs 1
   
   WARNING: this model evaluates AdaBoost detectors at every 4th pixel to speed up the computation. This results into overall performance
   of 68.7% PCP vs 69.2% PCP reported in the paper. To run slower dense classifiers, please, set the value of
   "window_desc_step_ratio" to "0.125" in abcparams_rounds500_dense05_2000.txt
   

   2) Train + Test
   
   a) local appearance model
   
	- train AdaBoost model for part <PARTIDX> out of 22 parts total
   	  
		<PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-local-app-model.txt --train_class --pidx <PARTIDX>
   
	- train generic spatial model
	
		<PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-local-app-model.txt --pc_learn
   
	- compute torso position prior
	
		<PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-local-app-model.txt --compute_pos_prior
   
	- train DPM part detectors (in matlab)

	   	cd <PARTAPP_DIR>/src/libs/libDPM
	   	trainDPM(<PARTIDX>, <EXP_DIR>)
	  
	  where <EXP_DIR> is the root of the experiments package
   
	- train DPM head detector (in matlab)
	
		cd <PARTAPP_DIR>/src/libs/libDPM
	   	trainDPM(11, <EXP_DIR>, true)
   
        - run local appearance model
	
		<PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-local-app-model.txt --head_detect_dpm --part_detect_dpm --find_obj 
   
   b) poselets

   	- run torso detector on training images

	      	<PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-train-torso.txt --find_obj --first <IMGIDX> --numimgs 1
   	
	- train poselet AdaBoost detectors
	   
		<PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-poselets.txt --train_class --pidx <PARTIDX> --tidx <TYPEIDX>

	  where <PARTIDX> is poselet id, 0-20; <TYPEIDX> is poselet type, 0-~100
   
	- collect poselet responses on training and testing images
	
		<PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-poselets.txt --save_resp_train --first <IMGIDX> --numimgs 1
	   	<PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-poselets.txt --save_resp_test --first <IMGIDX> --numimgs 1
   
	- train LDA classifiers
	
		<PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-poselets.txt  --train_lda_urot --train_lda_upos --train_lda_pwise
   
   c) full model
   
	- compute poselet conditioned mixtures
	
		<PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-full-model.txt --pc_learn_types
   
	- run full model
	
		<PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-full-model.txt --find_obj --first <IMGIDX> --numimgs 1

	- evaluate and visualize the results

   	  	<PARTAPP_DIR>/run_partapp.sh --expopt ./expopt/exp-lsp-full-model.txt --find_obj --first <IMGIDX> --numimgs 1

   3) Managing annotations
      
      Loading/saving annotation files in matlab:
      
      		cd <PARTAPP_DIR>/src/scripts/matlab
      
	        annotations = loadannotations(<annotations.al>);
		saveannotations(annotations, <annotations.al>);