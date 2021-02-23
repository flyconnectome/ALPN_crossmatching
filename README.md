# Matching neurons across data sets

## Background

Most insect neurons are morphological highly stereotyped: they look almost
exactly the same across animals - often down to medium size branches.
We also know that some neurons are clearly unique (i.e. there is only ever
one exemplar per hemisphere) while others come in multiple, highly similar
copies.

This knowledge is largely based on genetic driver lines that can be used to
label neurons with similar (but not necessarily known) genetic fingerprints
that then turn out to also have very similar morphology. Combined, these two
clues (genetics + morphology) provide strong support for declaring a "cell
type".

The advent of connectomes comes with a plethora of dense data: all of the
sudden we have morphologies for every single neuron neuron in the fly
brain - something that is next to impossible by genetic means.

Great! So let's just find similar looking neurons and make them a cell type.
Alas it's not that easy. Where do you make the cut off? How similar
do two neurons have to be, to be consider the same type? Can we find the same
groups across data sets?

That last point is of particular importance: we can define cell types based on
a single dataset - e.g. using confirmed types as landmarks - but they are
of limited use unless these types persist across animals. To address this, we
need to be able to identify neurons across data sets, and assess variability
and groupings to define sensible types.

Why we bother with this if it's so complicated? Because it's much easier to
think about a few thousand different cell types than about a hundred thousand
individual neurons.

## The data

Three complete sets of antennal lobe project neurons (ALPNs, ~335 neurons per
set) reconstructed in two different _Drosophila_ brains:
- FAFB (full adult fly brain) right hemisphere
- FAFB left hemisphere
- Janelia hemibrain ("FIB") right hemisphere

### Constraints

1. Neurons can only match within the same developmental (hemi-)lineage. This
   reduces the group size to ~90 in the worst case and to 1 in the best case.
   See `/data/meta.csv` for that information.

### Caveats

1. Every now and then a neuron has a developmental hiccup and does something
   unexpected. That can range from a missing or additional branch to taking a
   different tract. This can lead to bad scores between otherwise "correct"
   matches.
2. Some ALPNs are truncated in the hemibrain. We tried to compensate for this
   during the NBLAST but we currently don't factor in how much we loose due to
   truncation.
3. In some cases - typically when there are multiple sister neurons of the same
   type - there can be variations in numbers. Hence a 1:1:1 matching is not
   always possible.
4. For some neurons the lineage annotation might not be accurate. This is
   because the primary neurite bundles for some lineages (like l2PNs) do not
   tightly co-fasciculate, and some neurons can look ambiguous.   

### Files

`/data/nblast_scores.csv` contains an all-by-all NBLAST similarity score matrix.
See `/data/nblast.ipynb` for details on how these data were generated but in brief:

- FAFB and hemibrain (FIB) skeletons were transformed to the JRC2018F template brain
- left FAFB skeletons were additionally mirrored to the right
- small twigs (< 5 microns) were pruned off
- skeletons were resampled to 1 micron and turned into dotprops (points + tangent vectors)
- for FIB vs FAFB NBLASTs, the FAFB dotprops were truncated to the hemibrain volume
- for FAFB vs FAFB NBLASTs, the full dotprops were used
- scores were combined into one big score matrix

**Important**: These scores are normalized but not _symmetrized_. Typically we
use the mean of the forward and reverse scores which can be obtains like so:

```Python
>>> import pandas as pd
>>> scores = pd.read_csv('nblast_scores.csv')
>>> scores_mean = (scores + scores.T) / 2
```

`/data/meta.csv` contains meta data for each neuron:

- `id`: ID that match index/columns in `nblast_scores.csv`
- `name`: name of the neuron; this can be used as sanity check
- `lineage`: constraint for matching/typing
- `label`: labels of _known_ types; we expect mostly pairs/groups where labels match
- `is_canonical`: whether neuron labels are well established
- `ntype`: whether neuron is uni- or multiglomerular projection neuron
- `source`: to which dataset this neuron belongs

## References

The data provided in this repository is based on three publications:

- [Schlegel, Bates et al., bioRxiv (2020)](https://www.biorxiv.org/content/10.1101/2020.12.15.401257v2.full)
- [Bates, Schlegel et al., Current Biology (2020)](https://www.sciencedirect.com/science/article/pii/S0960982220308587)
- [Scheffer et al., eLife (2020)](https://elifesciences.org/articles/57443)

In particular, Figure 4 and S6 in Schlegel, Bates _et al._ represent our
attempts at cross-matching these ALPNs.
