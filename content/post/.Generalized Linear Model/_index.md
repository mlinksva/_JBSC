---
title: Generalized Linear Models
date: 2018-10-29
---

# Introduction

Generalized linear models are a framework which allows to model the relation between a linear predictor and a random component through a link function

$$ E(Y) = \mu = g^{-1}(X \beta) + u $$

where

- $ E(Y) = \mu $ is the expected value of a random variable with distribution $ F $ where $ F $ is a distribution of the exponential family

- $ g $ is the link function ($ g^{-1} $ is the inverse)

- $ X $ a set of features

- $ \beta $ is the parameter weights that combined with $ X $ make the linear predictor $ X \beta $

- $ u $ is the error term

The random component is fully specified by the outcomes and a distribution

The exponential family include the Normal, Multinomial with fixed number of trials, and Negative Binomial with fixed number of sucesses distributions

These distributions combined with various link functions allow for a flexible framework that allows to model continous, categorical (nominal and ordinal), count, and survival outcomes

# Example: Multinomial

## Motivation

Consider modeling a categorical outcome such as what activity to do tonight. Restrict the outcome space to watching a movie, go to a party, or sleep early. The outcome is nominal. Some variables that might be considered exogenous explanatory variables include the weather and the day of the week. This application is a probability model since it attempts to uncover the parameters that maximize the likelihood of the underlying probability distribution over the outcome space.

## Application

We can use the multinomial dataset provided from Germán Rodríguez (Princeton University) used in POP 507 / ECO 509 / WWS 509 - Generalized Linear Statistical Models course. The selected dataset is the "Contraceptive Use Data.

## Set Up

````julia
using Pkg: activate
activate("static/files/env/GLM")
using DataFrames: categorical!, DataFrame, head, stack
using Distributions: Distribution, Multinomial, logpdf
using StatsBase: coefnames, FrequencyWeights, RegressionModel, stderror
using StatsModels: @formula, ModelFrame, ModelMatrix, model_response
using LinearAlgebra: cholesky!, diag, Diagonal, Hermitian, LowerTriangular,
                     UpperTriangular, mul!, qr
using StatsFuns: softmax
using FisherScoring
````


<pre class="julia-error">
ERROR: ArgumentError: Package FisherScoring &#91;8ac8c9de-2d50-5604-b495-eeb5648d67fe&#93; is required but does not seem to be installed:
 - Run &#96;Pkg.instantiate&#40;&#41;&#96; to install all recorded dependencies.

</pre>


````julia
import StatsBase: coef, coefnames, CoefTable, dof, dof_residual, fit!, fitted,
                  loglikelihood, modelmatrix, nobs, r2, residuals, response,
                  vcov, weights
````





## Data

We will be using contraceptive use data for Fiji from the World Fertility Survey (Fiji Bureau of Statistics 1974).

````julia
data = DataFrame(age = repeat(["<25", "25-29", "30-39", "40-49"], inner = 4),
                 education = repeat(repeat(["low", "high"], inner = 2), 4),
                 wants_more = repeat(["yes", "no"], 8),
                 no = [53, 10, 212, 50, 60, 19, 155, 65, 112, 77, 118, 68, 35,
                       46, 8, 8],
                 yes = [6, 4, 52, 10, 14, 10, 54, 27, 33, 80, 46, 78, 6, 48, 12,
                        31]) |>
     categorical! |>
     (df -> stack(df, [:no, :yes],
                  variable_name = :use,
                  value_name = :count))
println(summary(data))
````


````
32×5 DataFrames.DataFrame
````



````julia
head(data, size(data, 1))
````



<table class="data-frame"><thead><tr><th></th><th>use</th><th>count</th><th>age</th><th>education</th><th>wants_more</th></tr><tr><th></th><th>Symbol</th><th>Int64</th><th>Categorical…</th><th>Categorical…</th><th>Categorical…</th></tr></thead><tbody><tr><th>1</th><td>no</td><td>53</td><td>&lt;25</td><td>low</td><td>yes</td></tr><tr><th>2</th><td>no</td><td>10</td><td>&lt;25</td><td>low</td><td>no</td></tr><tr><th>3</th><td>no</td><td>212</td><td>&lt;25</td><td>high</td><td>yes</td></tr><tr><th>4</th><td>no</td><td>50</td><td>&lt;25</td><td>high</td><td>no</td></tr><tr><th>5</th><td>no</td><td>60</td><td>25-29</td><td>low</td><td>yes</td></tr><tr><th>6</th><td>no</td><td>19</td><td>25-29</td><td>low</td><td>no</td></tr><tr><th>7</th><td>no</td><td>155</td><td>25-29</td><td>high</td><td>yes</td></tr><tr><th>8</th><td>no</td><td>65</td><td>25-29</td><td>high</td><td>no</td></tr><tr><th>9</th><td>no</td><td>112</td><td>30-39</td><td>low</td><td>yes</td></tr><tr><th>10</th><td>no</td><td>77</td><td>30-39</td><td>low</td><td>no</td></tr><tr><th>11</th><td>no</td><td>118</td><td>30-39</td><td>high</td><td>yes</td></tr><tr><th>12</th><td>no</td><td>68</td><td>30-39</td><td>high</td><td>no</td></tr><tr><th>13</th><td>no</td><td>35</td><td>40-49</td><td>low</td><td>yes</td></tr><tr><th>14</th><td>no</td><td>46</td><td>40-49</td><td>low</td><td>no</td></tr><tr><th>15</th><td>no</td><td>8</td><td>40-49</td><td>high</td><td>yes</td></tr><tr><th>16</th><td>no</td><td>8</td><td>40-49</td><td>high</td><td>no</td></tr><tr><th>17</th><td>yes</td><td>6</td><td>&lt;25</td><td>low</td><td>yes</td></tr><tr><th>18</th><td>yes</td><td>4</td><td>&lt;25</td><td>low</td><td>no</td></tr><tr><th>19</th><td>yes</td><td>52</td><td>&lt;25</td><td>high</td><td>yes</td></tr><tr><th>20</th><td>yes</td><td>10</td><td>&lt;25</td><td>high</td><td>no</td></tr><tr><th>21</th><td>yes</td><td>14</td><td>25-29</td><td>low</td><td>yes</td></tr><tr><th>22</th><td>yes</td><td>10</td><td>25-29</td><td>low</td><td>no</td></tr><tr><th>23</th><td>yes</td><td>54</td><td>25-29</td><td>high</td><td>yes</td></tr><tr><th>24</th><td>yes</td><td>27</td><td>25-29</td><td>high</td><td>no</td></tr><tr><th>&vellip;</th><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td></tr></tbody></table>



Consider the following data generation model

$$ use \sim age + education + wants_more $$

````julia
formula = @formula(use ~ age + education + wants_more)
````



````julia
mutable struct Model{A <: AbstractVecOrMat,
                     B <: AbstractVecOrMat,
                     C <: AbstractVecOrMat,
                     Dist <: Distribution,
                     Link <: AbstractLink,
                     W <: AbstractVector,
                     } <: RegressionModel
    X::Matrix
    y::B
    β::C
    Ψ::Hermitian
    D::Type{Dist}
    L::Link
    wts::W
    exposure_reference::NamedTuple
    varlist::NamedTuple
    function Model(formula, data, dist, link; wts = nothing)
        mf = ModelFrame(formula, data)
        y = model_response(mf)
        X = ModelMatrix(mf).m
        features = coefnames(mf)
        β = zeros()
end
modelmatrix(obj::Model) = obj.X
response(obj::Model) = obj.y
distribution(obj::Model) = obj.D
link(obj::Model) = obj.L
factorization(obj::Model) = qr(modelmatrix(obj))
weights(obj::Model) = obj.wts
exposure(obj::Model) = obj.offset_exposure.exposure
reference(obj::Model) = obj.offset_exposure.reference
````


<pre class="julia-error">
ERROR: syntax: incomplete: &quot;mutable&quot; at none:1 requires end
</pre>




## Struct

We can use a struct to keep things well organized

````julia
"""
    Model(formula, data) <: RegressionModel

    Use it for Multinomial logistic regression.
"""
mutable struct Model <: RegressionModel
    X::Matrix
    y::BitMatrix
    ω::FrequencyWeights
    β::Matrix
    varlist::NamedTuple
    function Model(formula, data; weights = :count)
        mf = ModelFrame(formula, data)
        y = model_response(mf) |>
            (obj -> mapreduce(val -> obj .== val, hcat, unique(obj)))
        outcomes = unique(model_response(mf))
        reference = 1
        categories = findall(!isequal(outcomes[reference]), outcomes)
        X = ModelMatrix(mf).m
        ω = FrequencyWeights(getproperty(data, weights))
        varlist = (reference = outcomes[reference],
                   outcome_space = string.(outcomes[categories]),
                   features = coefnames(mf),
                   categories = categories)
        return new(X, y, ω, float(zero(y)), varlist)
    end
end
model = Model(formula, data)
````





## Fisher Scoring

We now can use the Fisher Scoring algorithm to obtain the maximum likelihood estimates for the parameters of our model

To implement Fisher Scoring should

- Define the loglikelihood of the model
- Implement the link function
- Compute the QR decomposition of the model matrix
- Define the iteration protocol

### Fisher Scoring à la O'Leary

    Compute X = QR
    ℓℓ = [-Inf, -Inf]

    for iteration ∈ 1:max_iter
        ℓℓ₀ = loglikelihood(y, μ)
        η = μ + (y − g(μ)) / g′(μ)
        w = Diagonal(g′(μ) / var(g(μ)))
        β = inv(Q' * W * Q) * Q' * W * η
        μ = Q * β
        abs(ℓℓ(μ) - ℓℓ₀(μ)) < tol && break
    end

### Objective Function

````julia
loglikelihood(y::AbstractVector{<:Bool},
              μ::AbstractVector{<:Real},
              ω::Real) =
    ω * logpdf(Multinomial(1, μ), y)
loglikelihood(y::AbstractMatrix{<:Bool},
              μ::AbstractMatrix{<:Real},
              ω::AbstractVector{<:Real}) =
    sum(loglikelihood(y[idx,:], μ[idx,:], ω[idx]) for idx ∈ 1:size(y, 1))
````





### Mean (inverse link) function

We need to compute the mean, the derivative, and variance of the mean function (inverse of the link) at the

The cannonical link of the Multinomial distribution is the Logit link. The inverse of the Logit link function is the softmax function

$$ \mathbb{P}(y = j | x) = \frac{\exp(X \beta\_{j})}{\sum \exp(X\beta\_{j})} $$

When a distribution uses the cannonical link the derivative and the variance are the same.

````julia
function logit(η)
    g  = softmax(η)
    g′ = g .* (one.(g) .- g)
    μ, μ′, σ² = (g, g′, g′)
    return (μ, μ′, σ²)
end
function logit!(μ, μ′, σ², η)
    for idx ∈ 1:size(η, 1)
        @inbounds μ[idx,:], μ′[idx,:], σ²[idx,:] = @views logit(η[idx,:])
    end
end
````


````
logit! (generic function with 1 method)
````





### Updating η

````julia
function update_η!(Q, w, β, y, η, μ, μ′, categories)
    z = η + (y - μ) ./ μ′
    @views for idx ∈ categories
        C  = cholesky!(Hermitian(transpose(Q) * Diagonal(w[:,idx]) * Q)).factors
        β[:,idx]  = LowerTriangular(transpose(C)) \ (transpose(Q) * Diagonal(w[:,idx]) * z[:,idx])
        β[:,idx]  = UpperTriangular(C) \ β[:,idx]
    end
    return mul!(η, Q, β)
end
````


````
update_η! (generic function with 1 method)
````





### Initial Value

````julia
function initial_values(model)
    X, y, ω, categories = model.X, model.y, model.ω, model.varlist.categories
    F = qr(X, Val(true))
    all(elem -> abs(elem) > sqrt(eps()), diag(F.R)) ||
        throw(ArgumentError("Model Matrix is not full rank"))
    Q = Matrix(F.Q)
    m, n = size(X)
    β = zeros(n, size(y, 2))
    η = fill(inv(size(y, 2)), size(y))
    w = zero(η)
    μ = zero(η)
    μ′ = zero(η)
    σ² = zero(η)
    ℓℓ = [-Inf, -Inf]
    Δ = Inf
    tol = 1e-9
    iterations = 25
    return (X, y, ω, Q, categories, β, w, η, μ, μ′, σ², ℓℓ, Δ, tol, iterations)
end
````


````
initial_values (generic function with 1 method)
````



````julia
function update_ℓℓ!(ℓℓ, y, μ, w)
    ℓℓ .= (ℓℓ[2], loglikelihood(y, μ, ω))
    return ℓℓ
end
````


````
update_ℓℓ! (generic function with 1 method)
````



````julia
function update_z!(z, y, η, μ, μ′)
    z .= η + (y - μ) ./ μ′
end
````


````
update_z! (generic function with 1 method)
````



````julia
function update_w!(w, μ′, σ², ω)
    w .= abs2.(μ′) ./ σ² .* ω
end
````


````
update_w! (generic function with 1 method)
````



````julia
function fit!(model::Model)
    X, y, ω, Q, categories, β, w, η, μ, μ′, σ², ℓℓ, Δ, tol, iterations =
        initial_values(model)
    for itt ∈ 1:iterations
        logit!(μ, μ′, σ², η)
        update_ℓℓ!(ℓℓ, y, μ, w)
        update_w!(w, μ′, σ², ω)
        update_η!(Q, w, β, y, η, μ, μ′, categories)
        Δ = abs(reduce(-, ℓℓ)) / first(ℓℓ₀)
        if Δ < tol || iszero(first(ℓℓ))
            println(ℓℓ[2])
            model.β = β
            return model
        end
    end
    return 0
end
````


````
fit! (generic function with 4 methods)
````



````julia
fit!(model)
````


<pre class="julia-error">
ERROR: UndefVarError: ω not defined
</pre>




## Implement standard API

````julia
coef(obj::Model) = obj.β[:,obj.varlist.categories]
coefnames(obj::Model) = obj.varlist.features
nobs(obj::Model) = sum(obj.ω)
dof(obj::Model) = size(coef(obj), 1)
dof_residual(obj::Model) = nobs(obj) - dof(obj) - "(Intercept)" ∈ coefnames(obj)
modelmatrix(obj::Model) = obj.X
fitted(obj::Model) = modelmatrix(obj) * obj.β
response(obj::Model) = obj.y
residuals(obj::Model) = response(obj) - fitted(obj)
weights(obj::Model) = obj.ω
loglikelihood(obj::Model) = loglikelihood(response(obj),
                                          mapslices(softmax, fitted(obj), dims = 2),
                                          weights(obj))
function vcov(obj::Model)
end
````


````
vcov (generic function with 5 methods)
````



````julia
function obtain_Ω(X, μ, ω, reference)
    Xᵀ = transpose(X)
    k = size(μ, 2)
    p = size(X, 2)
    Σ = zeros(p * (k - 1), p * (k - 1))
    @views for (idx₀, col) ∈ enumerate(1:p:(p * k))
        if idx₀ != reference
            base_col = idx₀ > reference
            for (idx₁, row) ∈ enumerate(1:p:col)
                if idx₁ != reference
                    base_row = idx₁ > reference
                    rows = (row - base_row * p):(row - base_row * p + p - 1)
                    cols = (col - base_col * p):(col - base_col * p + p - 1)
                    if row == col
                        Σ[rows, cols] = Xᵀ * Diagonal(μ[:,idx₀] .* (1 .- μ[:,idx₀]) .* ω) * X
                    else
                        Σ[rows, cols] = Xᵀ * Diagonal(μ[:,idx₀] .* μ[:,idx₁] .* ω) * X
                    end
                end
            end
        end
    end
    Σ = Hermitian(Σ)
end
vcov(model::Model) = Hermitian(inv(cholesky!(obtain_Ω(model.X, μ, model.ω, model.reference))))
````



````julia
β = β[:,categories]
````


<pre class="julia-error">
ERROR: UndefVarError: β not defined
</pre>


````julia
Ψ = Hermitian(inv(cholesky!(obtain_Ω(X, μ, ω, reference))))
````


<pre class="julia-error">
ERROR: UndefVarError: X not defined
</pre>




# References

- Fiji Bureau of Statistics, International Statistical Institute. Fiji World Fertility Survey 1974. Voorburg, Netherlands: International Statistical Institute.
