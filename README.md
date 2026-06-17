function standard_ga_optimization()
    clear; clc; close all;
    
    %% ================== Watermark Configuration ==================
    % !! Modify your watermark text here !!
    watermark_text = 'CONFIDENTIAL - GA OPTIMIZATION'; 
    
    %% ================== 1. Genetic Algorithm Parameter Settings ==================
    pop_size = 80;         % Population size (number of individuals)
    chrom_length = 2;      % Chromosome length (number of variables: x1, x2)
    max_gen = 100;         % Maximum number of generations
    pc = 0.8;              % Crossover rate
    pm = 0.05;             % Mutation rate
    
    % Variable bounds [lower_bound, upper_bound]
    lb = -5.12; 
    ub = 5.12;
    
    %% ================== 2. Initialize Population ==================
    % Randomly generate a real-coded population within the defined bounds
    pop = lb + (ub - lb) * rand(pop_size, chrom_length);
    
    % Record the best and mean fitness of each generation for plotting
    best_fitness_history = zeros(1, max_gen);
    mean_fitness_history = zeros(1, max_gen);
    
    % Create visualization window
    figure('Name', 'Genetic Algorithm Optimization Live', 'Color', 'w', 'Position', [200 100 1200 550]);
    
    %% ================== 3. Genetic Algorithm Main Loop ==================
    for gen = 1:max_gen
        
        % --- A. Fitness Evaluation ---
        % For minimization problems: fitness = 1 / (objective_value + offset)
        % Lower objective value leads to higher fitness score
        obj_values = r_function(pop(:,1), pop(:,2));
        fitness = 1 ./ (obj_values + 1.0); 
        
        % Record current generation data
        [best_fit, best_idx] = max(fitness);
        best_solution = pop(best_idx, :);
        best_obj_val = obj_values(best_idx);
        
        best_fitness_history(gen) = best_obj_val;
        mean_fitness_history(gen) = mean(obj_values);
        
        % --- B. Selection Operation (Roulette Wheel Selection) ---
        fit_sum = sum(fitness);
        prob = fitness / fit_sum;      % Selection probability for each individual
        cum_prob = cumsum(prob);       % Cumulative probability roulette wheel
        
        new_pop = zeros(size(pop));
        for i = 1:pop_size
            r = rand();
            idx = find(cum_prob >= r, 1, 'first');
            new_pop(i, :) = pop(idx, :);
        end
        pop = new_pop;
        
        % --- C. Crossover Operation (Arithmetic Crossover) ---
        for i = 1:2:pop_size-1
            if rand() < pc
                % Randomly choose a crossover weight alpha
                alpha = rand();
                parent1 = pop(i, :);
                parent2 = pop(i+1, :);
                % Linear recombination
                pop(i, :) = alpha * parent1 + (1 - alpha) * parent2;
                pop(i+1, :) = (1 - alpha) * parent1 + alpha * parent2;
            end
        end
        
        % --- D. Mutation Operation (Uniform Mutation) ---
        for i = 1:pop_size
            for j = 1:chrom_length
                if rand() < pm
                    % Mutate randomly within bounds
                    pop(i, j) = lb + (ub - lb) * rand();
                end
            end
        end
        
        % Boundary Check Protection (Prevent mutation from exceeding the domain)
        pop(pop < lb) = lb;
        pop(pop > ub) = ub;
        
        % --- E. Live Animation Rendering ---
        if mod(gen, 2) == 0 || gen == 1
            % Pass lb and ub boundaries into the plotting function
            draw_plots(pop, gen, max_gen, best_solution, best_obj_val, best_fitness_history, watermark_text, lb, ub);
        end
    end
    
    %% ================== 4. Final Results Printing ==================
    fprintf('\n==============================================================\n');
    fprintf('  WATERMARK: [%s]\n', watermark_text);
    fprintf('================== GA Optimization Finished ==================\n');
    fprintf('Optimal Coordinates found: X1 = %.6f, X2 = %.6f\n', best_solution(1), best_solution(2));
    fprintf('Minimum Function Value (Global Best): %.6f\n', best_obj_val);
    fprintf('Theoretical Minimum is 0.000000 at (0,0)\n');
    fprintf('==============================================================\n');
end

%% ================== Subfunction: Rastrigin Objective Function ==================
function z = r_function(x1, x2)
    z = 20 + (x1.^2 - 10*cos(2*pi*x1)) + (x2.^2 - 10*cos(2*pi*x2));
end

%% ================== Subfunction: Dynamic Plotting ==================
% Function interface extended to accept lb and ub boundaries
function draw_plots(pop, gen, max_gen, best_sol, best_val, history, txt, lb, ub)
    clf;
    
    % ---- Left Subplot: 3D Landscape & Population Distribution ----
    subplot(1, 2, 1);
    [X, Y] = meshgrid(linspace(lb, ub, 50)); % Hardcoded limits replaced with lb and ub variables
    Z = 20 + (X.^2 - 10*cos(2*pi*X)) + (Y.^2 - 10*cos(2*pi*Y));
    surf(X, Y, Z, 'EdgeColor', 'none', 'FaceAlpha', 0.6);
    colormap('jet'); hold on;
    
    % Calculate the elevation of current population
    pop_z = 20 + (pop(:,1).^2 - 10*cos(2*pi*pop(:,1))) + (pop(:,2).^2 - 10*cos(2*pi*pop(:,2)));
    % Plot regular individuals currently searching (red scatter points)
    scatter3(pop(:,1), pop(:,2), pop_z, 25, 'r', 'filled');
    % Plot the best individual of current generation (large green pentagram)
    best_z = 20 + (best_sol(1)^2 - 10*cos(2*pi*best_sol(1))) + (best_sol(2)^2 - 10*cos(2*pi*best_sol(2)));
    plot3(best_sol(1), best_sol(2), best_z, 'gp', 'MarkerSize', 15, 'MarkerFaceColor', 'g');
    
    view(35, 45); grid on;
    xlabel('X1'); ylabel('X2'); zlabel('F(X1, X2)');
    title(sprintf('Generation: %d / %d | Population Distribution', gen, max_gen));
    
    % [Watermark for Left Plot] - Dynamically adapts to lb and ub values
    text(lb, ub, 80, txt, 'FontSize', 12, 'FontWeight', 'bold', ...
         'Color', [0.5 0.5 0.5], 'HorizontalAlignment', 'left', 'Clipping', 'off');
    
    % ---- Right Subplot: Convergence Curve ----
    subplot(1, 2, 2);
    plot(1:gen, history(1:gen), 'b-', 'LineWidth', 2);
    grid on;
    xlabel('Generation'); ylabel('Best Function Value');
    title(sprintf('Convergence Curve (Current Best: %.4f)', best_val));
    xlim([1 max_gen]);
    
    % [Watermark for Right Plot] - Semi-transparent looking text at the center
    xl = xlim; yl = ylim;
    text(xl(1) + (xl(2)-xl(1))*0.5, yl(1) + (yl(2)-yl(1))*0.5, txt, ...
         'FontSize', 16, 'FontWeight', 'bold', 'Color', [0.85 0.85 0.85], ...
         'HorizontalAlignment', 'center', 'VerticalAlignment', 'middle', 'Rotation', 15);
    
    drawnow;
end
