%% Script to loop through entire folder of datafiles                                                           COMMENTS AND EXPLANATIONS
                                                                                                                 
% Restore MATLAB default path and add necessary paths                                                           
    restoredefaultpath
    addpath 'C:\Users\melis\Documents\MATLAB\fieldtrip-20240731'
    addpath 'C:\Users\melis\Documents\MATLAB\Scripts'
    ft_defaults
    
    % Define the directory containing the BrainVision files
    data_dir                    = 'C:\Users\melis\Documents\Trento\Verona_Experiment\Prova01_10'; 
    preproc_dir                 = fullfile(data_dir, 'Preprocessed');                                           % Directory to save preprocessed files
        if ~exist(preproc_dir, 'dir')
            mkdir(preproc_dir);                                                                                 % Create output directory if it doesn't exist
        end

%% First Loop: Load, preprocess, and save filtered data

dbstop if error
    file_list                   = dir(fullfile(data_dir, '*.vhdr'));                                            % Get a list of all .vhdr files
    
    for file_idx                = 1:length(file_list)
        
        % Get the current .vhdr file and construct associated file paths
        vhdr_file               = fullfile(data_dir, file_list(file_idx).name);                                 % Full path to .vhdr file
        [~, base_name, ~]       = fileparts(vhdr_file);
        eeg_file                = fullfile(data_dir, [base_name, '.eeg']);                                      % Full path to corresponding .eeg file
    
        % Extract the base name and replace underscores with spaces
        name                    = strrep(base_name, '_', ' ');                                                  % Replace '_' with ' ' in the base file name
        fprintf('Processing file (loading and preprocessing): %s (Name: %s)\n', vhdr_file, name);               % Debugging output
            
        
            %%% PREPROCESSING STEPS:
            
            % segment data into trials with correct trialfunction
            cfg                 = [];
            cfg.trialfun        = 'trialfun_visgam_VR'; 
            cfg.headerfile      = vhdr_file;                                                                    % Use dynamically assigned vhdr_file
            cfg.datafile        = eeg_file;                                                                     % Use dynamically assigned eeg_file
            trialdata           = ft_definetrial(cfg);
            
            
            % preprocessing and rereferencing
            cfg.implicitref     = 'Fp1';
            cfg.reref           = 'yes';
            cfg.refchannel      = 'average';
            cfg.dftfreq         = [50 100 150];
            preproc_data        = ft_preprocessing(trialdata);
            
            
            % resampling
            cfg                 = [];
            cfg.resamplefs      = 300;
            resampdata          = ft_resampledata(cfg, preproc_data);
    
    
        % Save the preprocessed data
        save(fullfile(preproc_dir, [base_name, '_preproc.mat']), 'resampdata');
    
    
    end
%% Second Loop: Load preprocessed data and Clean

dbstop if error
    % If necessary, define the directory containing the preprocessed files
    %data_dir                   ='your path';                                                                 
    preproc_dir                = fullfile(data_dir, 'Preprocessed');                                           
                                                                                                                
    file_list                   = dir(fullfile(preproc_dir, '*.mat'));                                          % Get a list of all .mat files
    
    for file_idx                = 1:length(file_list)
        
        % Get the current file's base name
        [~, base_name, ~]       = fileparts(file_list(file_idx).name);
    
        % Load the preprocessed data
        preproc_file            = fullfile(preproc_dir, [base_name, '.mat']);
            if ~isfile(preproc_file)
                fprintf('Preproc file not found for %s. Skipping.\n', base_name);
                continue;
            end
        load(preproc_file, 'resampdata');                                                                       % Load saved filtered_data
        fprintf('Processing file (Cleaning): %s\n', base_name);                                                 % Debugging output
         
    
            %%% CLEANING STEPS:
    
            % visual artifact detection
            cfg                 = [];
            cfg.demean          = 'yes';
            cfg.detrend         = 'yes';
            cfg.ylim            = [-30 30];
            brodata             = ft_databrowser(cfg, resampdata);
            
            % visual artifact rejection
            cfg                                 = [];                                                           
            cfg.artfctdef.visual.artifact       = brodata.artfctdef.visual.artifact;
            cfg.artfctdef.reject                = 'complete';
            data_clean                         = ft_rejectartifact(cfg, resampdata);
            
            % % redefine trial after artifact rejection 
            % cfg                 =[];
            % cfg.toilim          = [-1.0 3.0];
            % cfg.minlength       = 'maxperlen'; 
            % data_clean          = ft_redefinetrial(cfg, data_artrem);
           
            % identify bad channels
            cfg                 = [];
            cfg.demean          = 'yes';
            cfg.detrend         = 'yes';
            cfg.ylim            = [-30 30];
            ft_databrowser(cfg, data_clean)
            badchannel          = input('badchannel');                                                          % {'Fp1', 'Fp2', '...', '...'}, always put ground and ref channel in here too
            
            % prepare neighbours for channel repair
            cfg = [];
            cfg.channel         = 'all' ;
            cfg.method          = 'triangulation';
            cfg.template        = 'easycap64ch-avg_neighb.mat';
            cfg.layout          = 'easycapM11.mat';
            neighbours          = ft_prepare_neighbours(cfg, data_clean);
            
            % repair bad channels
            cfg = [];
            cfg.badchannel      = badchannel;
            cfg.method          = 'average';
            cfg.neighbours      = neighbours;
            data_fixed          = ft_channelrepair(cfg,data_clean);
    
    
        % Save the cleaned data
        cleaned_dir             = fullfile(data_dir, 'Cleaned');                                                % Directory to save cleaned files
            if ~exist(cleaned_dir, 'dir')
                mkdir(cleaned_dir);                                                                             % Create output directory if it doesn't exist
            end
        name                    = strrep(base_name, 'preproc', 'cleaned');                                      % Replace 'preproc' with 'cleaned' in filename  
        save(fullfile(cleaned_dir, name), 'data_fixed', '-v7.3');                                                        % to prevent filenames from getting too long
    
    end

            %%% In case you want to check if the channel repair worked
            % cfg                 = [];
            % cfg.demean          = 'yes';
            % cfg.detrend         = 'yes';
            % cfg.ylim            = [-30 30];
            % ft_databrowser(cfg, data_fixed)
%% Third Loop: ICA

dbstop if error
    % If necessary, define the directory containing the preprocessed files
    % data_dir                  ='C:\Users\melis\Documents\Trento\Verona_Experiment\YS.visGamma.20241029';     % these two lines only need to be executed if 
    % cleaned_dir               = fullfile(data_dir, 'Cleaned');                                               % you want to resume your analysis here after 
                                                                                                               % closing matlab or losing your workspace
    file_list                   = dir(fullfile(cleaned_dir, '*.mat'));                                         % Get a list of all .mat files
    
    for file_idx                = 1:length(file_list)
        
        % Get the current file's base name
        [~, base_name, ~]       = fileparts(file_list(file_idx).name);
    
        % Load the cleaned data
        cleaned_file            = fullfile(cleaned_dir, [base_name, '.mat']);
            if ~isfile(cleaned_file)
                fprintf('Cleaned file not found for %s. Skipping.\n', base_name);
                continue;
            end
        load(cleaned_file, 'data_fixed');                                                                       % Load saved filtered_data
        fprintf('Processing file (ICA): %s\n', base_name);                                                      % Debugging output        
            
            %%% ICA STEPS:

            % ICA decomposition
            cfg                 = [];
            cfg.method          = 'fastica';
            data_comp           = ft_componentanalysis(cfg, data_fixed);                                        % using the data without atypical artifacts
            
            
            % Identifying artifactual components
            cfg                 = [];
            cfg.layout          = 'acticap-64ch-standard2.mat';  
            cfg.marker          = 'off';                            
            
                % First window: Components 1–20
                cfg.component       = 1:20;                                                                         % Select components 1–21
                figure('Name', 'Components 1–20', 'NumberTitle', 'off');                                            % Create a new figure
                ft_topoplotIC(cfg, data_comp);                                                                      % Plot the first group of components
                ft_databrowser(cfg, data_comp)
                % Second window: Components 21–40
                cfg.component       = 21:40;                                                                        % Select components 22–42
                figure('Name', 'Components 21–40', 'NumberTitle', 'off');                                           % Create a new figure
                ft_topoplotIC(cfg, data_comp);                                                                      % Plot the second group of components
                
                % Third window: Components 41 to last component
                last_component      = size(data_comp.topo, 2);                                                      % Determine the total number of components
                cfg.component       = 43:last_component;                                                            % Select components 43 to the last one
                figure('Name', sprintf('Components 43-%d', last_component), 'NumberTitle', 'off');                  % Create a new figure
                ft_topoplotIC(cfg, data_comp);                                                                      % Plot the third group of components

             
            % remove the bad components                                                                         % Best to close the topoplots before hitting Enter for the bad 
            cfg                 = [];                                                                           % components, so topoplots of different datasets don't pile up
            cfg.component       = input('badcomponents');                                                       % badcomponents = [.., ...]
            data_postica        = ft_rejectcomponent(cfg, data_comp, data_fixed );
            

        % Save the cleaned data
        postica_dir             = fullfile(data_dir, 'PostICA');                                                % Directory to save cleaned files
            if ~exist(postica_dir, 'dir')
                mkdir(postica_dir);                                                                             % Create output directory if it doesn't exist
            end
        name                    = strrep(base_name, 'cleaned', 'postica');                                      % Replace 'cleaned' with 'postica' in filename  
        save(fullfile(postica_dir, name), 'data_postica', '-v7.3');                                                      % to prevent filenames from getting too long
                                                                                                                
    end

    
    %% In case you want to check 
            % cfg                 = [];
            % cfg.demean          = 'yes';
            % cfg.detrend         = 'yes';
            % cfg.ylim            = [-30 30];
            % ft_databrowser(cfg, data_postica)

%% Fourth Loop: Time-Frequency-Analysis
    
dbstop if error
    % If necessary, define the directory containing the preprocessed files
    % data_dir                  ='C:\Users\melis\Documents\Trento\Verona_Experiment\YS.visGamma.20241029';      % these two lines only need to be executed if 
    postica_dir               = fullfile(data_dir, 'PostICA');                                                % you want to resume your analysis here after 
                                                                                                                % closing matlab or losing your workspace
    file_list                   = dir(fullfile(postica_dir, '*.mat'));                                          % Get a list of all .mat files
    
    for file_idx = 1:length(file_list)
        
        % Get the current file's base name
        [~, base_name, ~]       = fileparts(file_list(file_idx).name);
    
        % Load the cleaned data
        postica_file            = fullfile(postica_dir, [base_name, '.mat']);
            if ~isfile(postica_file)
                fprintf('PostICA file not found for %s. Skipping.\n', base_name);
                continue;
            end
        load(postica_file, 'data_postica');                                                                     % Load saved filtered_data
        fprintf('Processing file (TFR): %s\n', base_name);                                                      % Debugging output        
            
            %%% TIME-FREQUENCY-ANALYSIS STEPS:

            %Filtering
            cfg = [];
            cfg.channel         = (1:31);
            cfg.detrend         = 'yes';
            cfg.demean          = 'yes';
            cfg.dftfilter       = 'yes';
            cfg.dftfreq         = [50, 100];
            cfg.baseline        = [-1 -0.05];
            data_preproc2       = ft_preprocessing(cfg,data_postica);
            
            
            % %Redefinetrials
            % cfg                 = [];
            % cfg.toilim          = [-1.0 3.0];
            % cfg.minlength       = 'maxperlen';                                                                  % ensures all resulting trials are equal length
            % data_stim           = ft_redefinetrial(cfg, data_preproc2);
            
            
            % TFR
            cfg                 = [];
            cfg.keeptrials      = 'yes';
            cfg.output          = 'pow';
            cfg.method          = 'mtmconvol';
            cfg.taper           = 'hanning';
            cfg.tapsmofrq       =   5;
            cfg.channel         = 'all';
            cfg.foi             = 30:1:80;                                                                      % set frequencies of interest
            cfg.toi             = -1:0.05:3;                                                                  % set timepoints of interest
            cfg.t_ftimwin       = 0.45 * ones(length(cfg.foi), 1);                                              % set time window for TFR analysis: constant length of 200ms
            cfg.pad             = 5;                                                                            % pad trials to integer number of seconds, this speeds up the 
            freq_stim           = ft_freqanalysis(cfg, data_preproc2);                                              % analysisand results in a neatly spaced frequency axis


        % Save the cleaned data
        TFR_dir                 = fullfile(data_dir, 'TFR');                                                    % Directory to save cleaned files
            if ~exist(TFR_dir, 'dir')
                mkdir(TFR_dir);                                                                                 % Create output directory if it doesn't exist
            end
        name                    = strrep(base_name, 'postica', 'TFR');                                          % Replace 'postica' with 'TFR' in filename  
        save(fullfile(TFR_dir, name), 'freq_stim');                                                             % to prevent filenames from getting too long
                                                                                                                
    end

%% Fifth Loop: Plot and save figures

 dbstop if error
    % If necessary, define the directory containing the preprocessed files
    % data_dir                  ='C:\Users\melis\Documents\Trento\Verona_Experiment\YS.visGamma.20241029';      % these two lines only need to be executed if 
    TFR_dir                   = fullfile(data_dir, 'TFR');                                                    % you want to resume your analysis here after 
                                                                                                                % closing matlab or losing your workspace
    file_list                   = dir(fullfile(TFR_dir, '*.mat'));                                              % Get a list of all .mat files
    
    for file_idx = 1:length(file_list)
        
        % Get the current file's base name
        [~, base_name, ~]       = fileparts(file_list(file_idx).name);
    
        % Load the cleaned data
        TFR_file                = fullfile(TFR_dir, [base_name, '.mat']);
            if ~isfile(TFR_file)
                fprintf('PostICA file not found for %s. Skipping.\n', base_name);
                continue;
            end
        load(TFR_file, 'freq_stim');                                                                            % Load saved filtered_data
        fprintf('Processing file (TFR): %s\n', base_name);                                                      % Debugging output        

            %%% PLOT FREQUENCIES
            
            % Questionable Layout Creation 
            cfg                 = [];
            cfg.layout          = 'easycapM1.mat';
            %layout.label       = hdr.label';
            %layout.pos         = 'standard_1020.elc';
            layout              = ft_prepare_layout(cfg);
            
            % Plot
            cfg = [];
            cfg.baseline        = [-1 0]; 
            cfg.baselinetype    = 'relative';
            %cfg.zlim           = [-3e-27 3e-27];
            cfg.xlim            = [0 3];
            cfg.ylim            = [30 80]; 
            cfg.showlabels      = 'yes';	
            cfg.layout          = layout;
            cfg.colorbar        = 'yes';
            %ft_topoplotTFR(cfg, freq_stim);
            ft_multiplotTFR(cfg, freq_stim);% Unique figure for each file
            %ft_singleplotTFR(cfg, freq_stim)
            title(strrep(base_name, '_', ' '))


        % Save all figures to a Figures folder
        figures_dir = fullfile(data_dir, 'Figures');                                                            % Define the Figures folder path
        if ~exist(figures_dir, 'dir')
            mkdir(figures_dir);                                                                                 % Create the folder if it doesn't exist
        end

        figure_file = fullfile(figures_dir, [base_name, '.fig']);                                               % Define the .fig file path
        figure(file_idx);                                                                                       % Select the current figure
        savefig(figure_file);                                                                                   % Save the figure
        fprintf('Figure for %s saved to %s\n', base_name, figure_file);                                         % Log success
                                                                                                                      
    end
