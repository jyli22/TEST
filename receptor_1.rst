.. _ReceptorSchema:

Receptor Schema
==============================

The purpose of the ``Receptor`` object is to represent reactivity data
of actual *Receptors*, i.e., Ig or TCR, either directly or indirectly.
To this end, the ``Receptor`` object has two main sections:

1. ``receptor_*`` properties: These properties describe the receptor
   as an abstract and global concept, i.e., the actual Ig/TCR protein
   complex MAY or MAY NOT have been observed in the current study.
   However, the Rearrangements encoding the respective chains MUST
   be present in the studies as well as the information linking them
   (see below). This allows data curators to provide potential
   reactivities of a potential receptor by referencing to its entry in
   an external database (e.g., IEDB).
2. ``reactivity_measurement`` array: This structure contains
   one or more entries, describing reactivities of the receptor that
   have been observed in the current study.

The ``Receptor`` object explicitly requires full sequence information
of the two associated variable domains. This is considered to be an
acceptable restriction from an AIRR-seq perspective, where sequencing
typically precedes or takes place in combination with the determination
of receptor reactivity.


Identifiers
-----------

The ``Receptor`` objects has two properties that serve as identifiers:

*  ``receptor_id`` is a **local** identifier and its uniqueness MUST NOT
   be assumed beyond the scope of the study the receptor was reported
   in. This property can be used, e.g., to represent designations for
   Ig/TCR used in a manuscript.
*  ``receptor_hash`` is the SHA256 hash of the receptors variable domain
   amino acid sequences, which serves as a **globally unique**
   identifier that can be independently calculated by repositories
   without requiring prior communication. It is calculated as follows,
   where ``base16`` designates the function described in `RFC4648
   Section 6`_:

   .. code-block::

      lower_case(
          base16(
              sha256(
                  concatenate(
                      upper_case(receptor_variable_domain_1_aa),
                      upper_case(receptor_variable_domain_2_aa)
                  )
              )
          )
      )


Relations to other AIRR Schema objects
--------------------------------------

The ``Receptor`` object is only directly linked to the ``Cell`` object,
which then in turn contains the references to the records in the
``Rearrangements`` that encode the respective chains of the receptor.
Therefore a given rearrangement cannot directly reference to a receptor,
which is also not a meaningful thing to do, as the paired chain would
be unclear, but is necessary to determine a receptors reactivity.


Annotation guidelines
---------------------

Reactivity information SHOULD be provided if it was either directly
recorded in the current study or is indirectly available in a public
database, where it can be linked. If no more information about the receptor,
besides the translated amino acid sequence of the associated chains is
available, the respective object MAY still be created if the curator
considers this information to be beneficial, e.g., if there is some
evidence (surface expression, reaction to superantigens) showing the
receptor is functional and present on the surface. In the absence of any
such data, ``Receptor`` objects SHOULD NOT be created.


Note on cells expressing more than a single receptor
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Cells that express more than a single IGH/TRB/TRD or a single
IGK/IGL/TRA/TRG chain are regularly observered as allelic exclusion is
never complete and its efficiency is rather low for loci like TRA.
Such dual-expressing cells can technically be accommodated in the
current AIRR Schema as an individual ``Cell`` object can link to more
than two rearrangemts and to more than a single ``Receptor``. In the
case of two potential receptors, both MAY be created as an objects, if
the general annotation rules are met (i.e., the direct or indirect
presence of reactivity information) for each of them. Note that the 
annotation of **cell**-based reactivity information is currently not
supported for dual-expressing cells, in this case additional information
confirming the reactivity of one of the receptors would be required. 


Representation of bi-specific antibodies
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The goal of the AIRR Standards is primarily to represent naturally
occuring receptors. While bi-specific antibodies may arise in
dual IGK/IGL expressing B cells their individual reactivity is
not measured on a regular basis. Therefore they are currently not
supported in the ``Receptor`` schema.


Reactivity Measurement Fields
-----------------------------

A ``reactivity_measurement`` array records the experimental design and result for what a researcher measured 
or observed for the reactivity of target ``Receptor``. Specifically, a receptor may have multiple reactivity 
measurement records, i.e. under a ``Receptor`` object there may be multiple measurement sets within the 
``reactivity_measurement`` array. All properties within one measurement set are specific to a single reactivity 
measurement record so that target receptors could be acquired by searching for the appropriate ``reactivity_measurement`` fields. 
The specification of these fields are all optional, which defines that there are no requirements for column 
appearing in the TSV. Besides that, regarding the variety of structure interaction between epitope and receptors 
of the immune system (BCR, TCR and MHC), several fields are nullable by assigning an empty string as the value.

Within ``reactivity_measurement`` array, it is expected that ``antigen_source_species``, ``peptide``, ``peptide_start`` 
and ``peptide_end`` properties have an inseparable relationship with ``antigen_type``. They only present a valid value 
when ``antigen_type`` is **protein** or **peptide**, otherwise they are NULL value. In the former case, ``peptide`` should 
present the actual peptide sequence, and ``antigen`` field would require the reference protein sequence of the 
experiment-measured peptide, to which ``peptide`` refers. In the meanwhile, ``peptide_start`` and ``peptide_end`` indicate 
the location of the actual tested peptide in reference sequence. There is a unique mapping between ``peptide`` and reference 
sequence acquired in ``antigen`` field. For example, peptide "RNVDENANANSAVKN" and "PNANPNVDPNANPNV" were used to assess 
receptor reactivity, in measurement records they are all of same antigen, Circumsporozoite protein (NF54), yet located in 
different sections, former from 291 to 305 while latter from 105 to 119. The same issue can also be expected in ``mhc_*`` 
properties. These fields specifically presents for MHC:x ligand types, i.e. if ``ligand_type`` is **MHC:peptide** or **MHC:non-peptide**,
``mhc_*`` fields would capture the MHC class and molecule of epitope and list the alleles identified in an individual. 
If requiring ``reactivity_measurement`` for B Cell Receptors, the ``mhc_*`` fields shoule be NULL value.


.. _ReceptorFields:

Receptor Fields
-----------------------------

:download:`Download as TSV <../_downloads/Receptor.tsv>`

.. list-table::
    :widths: 20, 15, 15, 50
    :header-rows: 1

    * - Name
      - Type
      - Attributes
      - Definition
    {%- for field in Receptor_schema %}
    * - ``{{ field.Name }}``
      - {{ field.Type }}
      - {{ field.Attributes }}
      - {{ field.Definition | trim }}
    {%- endfor %}

.. === References and Links ===

.. _`RFC4648 Section 6`: https://datatracker.ietf.org/doc/html/rfc4648#section-6
