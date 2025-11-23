# Trabalho Final FPI

Autores:
- Breno Morais
- Leonardo Martins

### Pseudo-Código
```Python
function Colorize(gray_image, scribbles):
    # Dimensions
    H, W = dimensions(gray_image)
    N = H * W  # Total pixels
    
    # Initialize Sparse Matrix A (size N x N)
    # Initialize Vectors b_u and b_v (size N)
    
    # Convert constraints to YUV
    constraints_YUV = rgb_to_yuv(scribbles)

    FOR each pixel r in image:
        row_index = index(r)
        
        IF pixel r is scribbled (constrained):
            # Hard constraint: U(r) = known_value
            A[row_index, row_index] = 1.0
            b_u[row_index] = constraints_YUV[r].u
            b_v[row_index] = constraints_YUV[r].v
            
        ELSE:
            # Optimization constraint: Color equals weighted avg of neighbors
            A[row_index, row_index] = 1.0
            b_u[row_index] = 0
            b_v[row_index] = 0
            
            neighbors = get_spatial_neighbors(r) # usually 3x3 window
            weights = []
            
            # Calculate weights based on Intensity (Y) similarity
            FOR each neighbor s in neighbors:
                # Equation 2 or 3 from paper
                diff = (gray_image[r] - gray_image[s])^2
                variance = local_variance(gray_image, r)
                w_rs = exp(-diff / (2 * variance))
                weights.append(w_rs)
            
            # Normalize weights so they sum to 1
            sum_weights = sum(weights)
            weights = weights / sum_weights
            
            # Fill matrix A with negative weights for neighbors
            FOR i, s in enumerate(neighbors):
                col_index = index(s)
                A[row_index, col_index] = -weights[i]

    # Solve linear systems using a sparse solver
    # Solves Ax = b
    final_U = sparse_solve(A, b_u)
    final_V = sparse_solve(A, b_v)
    
    # Merge channels
    final_image_yuv = merge_channels(gray_image, final_U, final_V)
    final_image_rgb = yuv_to_rgb(final_image_yuv)
    
    RETURN final_image_rgb
```

