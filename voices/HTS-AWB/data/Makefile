# ----------------------------------------------------------------- #
#           The HMM-Based Speech Synthesis System (HTS)             #
#           developed by HTS Working Group                          #
#           http://hts.sp.nitech.ac.jp/                             #
# ----------------------------------------------------------------- #
#                                                                   #
#  Copyright (c) 2001-2012  Nagoya Institute of Technology          #
#                           Department of Computer Science          #
#                                                                   #
#                2001-2008  Tokyo Institute of Technology           #
#                           Interdisciplinary Graduate School of    #
#                           Science and Engineering                 #
#                                                                   #
# All rights reserved.                                              #
#                                                                   #
# Redistribution and use in source and binary forms, with or        #
# without modification, are permitted provided that the following   #
# conditions are met:                                               #
#                                                                   #
# - Redistributions of source code must retain the above copyright  #
#   notice, this list of conditions and the following disclaimer.   #
# - Redistributions in binary form must reproduce the above         #
#   copyright notice, this list of conditions and the following     #
#   disclaimer in the documentation and/or other materials provided #
#   with the distribution.                                          #
# - Neither the name of the HTS working group nor the names of its  #
#   contributors may be used to endorse or promote products derived #
#   from this software without specific prior written permission.   #
#                                                                   #
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND            #
# CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES,       #
# INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF          #
# MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE          #
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS #
# BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,          #
# EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED   #
# TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,     #
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON #
# ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,   #
# OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY    #
# OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE           #
# POSSIBILITY OF SUCH DAMAGE.                                       #
# ----------------------------------------------------------------- #

# setting
SPEAKER = awb
DATASET = arctic

# awk and perl
AWK  = gawk
PERL = /usr/bin/perl

# SPTK commands
X2X     = /usr/local/SPTK/bin/x2x
MGCEP   = /usr/local/SPTK/bin/mcep
LPC2LSP = /usr/local/SPTK/bin/lpc2lsp
MERGE   = /usr/local/SPTK/bin/merge
SOPR    = /usr/local/SPTK/bin/sopr
NAN     = /usr/local/SPTK/bin/nan
MINMAX  = /usr/local/SPTK/bin/minmax
PITCH   = /usr/local/SPTK/bin/pitch
FRAME   = /usr/local/SPTK/bin/frame
WINDOW  = /usr/local/SPTK/bin/window
INTERPOLATE  = /usr/local/SPTK/bin/INTERPOLATE
DS  = /usr/local/SPTK/bin/DS
CH_WAVE  = /home/nlphacker/work/OpenSpeech/tools/speech_tools/bin/ch_wave

# dumpfeats to extract utterance information
DUMPFEATS = /home/nlphacker/work/OpenSpeech/tools/festival/examples/dumpfeats

# speech analysis conditions
SAMPFREQ   = 48000   # Sampling frequency (48kHz)
FRAMELEN   = 1200   # Frame length in point (1200 = 48000 * 0.025)
FRAMESHIFT = 240 # Frame shift in point (240 = 48000 * 0.005)
WINDOWTYPE = 1 # Window type -> 0: Blackman 1: Hamming 2: Hanning
NORMALIZE  = 1  # Normalization -> 0: none  1: by power  2: by magnitude
FFTLEN     = 2048     # FFT length in point
FREQWARP   = 0.55   # frequency warping factor
GAMMA      = 0      # pole/zero weight for mel-generalized cepstral (MGC) analysis
MGCORDER   = 34    # order of MGC analysis
LNGAIN     = 1     # use logarithmic gain rather than linear gain
LOWERF0    = 50    # lower limit for f0 extraction (Hz)
UPPERF0    = 280    # upper limit for f0 extraction (Hz)

# windows for calculating delta features
MGCWIN  = win/mgc.win
LF0WIN  = win/lf0.win
NMGCWIN = 3
NLF0WIN = 3

all: analysis labels

analysis: mgc lf0 cmp

labels: label mlf list scp

mgc:
	# Convert wav to raw
	for wav in wav/$(DATASET)_$(SPEAKER)_*.wav; do \
		base=`basename $${wav} .wav`; \
		$(CH_WAVE) -c 0 -F 32000 -otype raw $${wav} | $(X2X) +sf | $(INTERPOLATE) -p 2 -d | $(DS) -s 43 | $(SOPR) -m 0.69 | $(X2X) +fs > $${base}.raw; \
	done

	# Extracting MGC or MGC-LSP coefficients from raw audio
	mkdir -p mgc

	SAMPKHZ=`echo $(SAMPFREQ) | $(X2X) +af | $(SOPR) -m 0.001 | $(X2X) +fa`; \
	for raw in raw/$(DATASET)_$(SPEAKER)_*.raw; do \
		base=`basename $${raw} .raw`; \
		min=`$(X2X) +sf $${raw} | $(MINMAX) | $(X2X) +fa | head -n 1`; \
		max=`$(X2X) +sf $${raw} | $(MINMAX) | $(X2X) +fa | tail -n 1`; \
		if [ -s $${raw} -a $${min} -gt -32768 -a $${max} -lt 32767 ]; then \
			$(X2X) +sf $${raw} > tmp; \
			if [ $(GAMMA) -eq 0 ]; then \
				echo "Extracting MGC coefficients from $${raw}"; \
				$(FRAME) -l $(FRAMELEN) -p $(FRAMESHIFT) tmp | \
				$(WINDOW) -l $(FRAMELEN) -L $(FFTLEN) -w $(WINDOWTYPE) -n $(NORMALIZE) | \
				$(MGCEP) -a $(FREQWARP) -m $(MGCORDER) -l $(FFTLEN) -e 1.0E-08 > mgc/$${base}.mgc; \
			else \
				echo "Extracting MGC-LSP coefficients from $${raw}"; \
				if [ $(LNGAIN) -eq 1 ]; then \
					GAINOPT="-l"; \
				fi; \
				$(FRAME) -l $(FRAMELEN) -p $(FRAMESHIFT) tmp | \
				$(WINDOW) -l $(FRAMELEN) -L $(FFTLEN) -w $(WINDOWTYPE) -n $(NORMALIZE) | \
				$(MGCEP) -a $(FREQWARP) -c $(GAMMA) -m $(MGCORDER) -l $(FFTLEN) -e 1.0E-08 -o 4 | \
				$(LPC2LSP) -m $(MGCORDER) -s $${SAMPKHZ} $${GAINOPT} -n $(FFTLEN) -p 8 -d 1.0E-08 > mgc/$${base}.mgc; \
			fi; \
			if [ -n "`$(NAN) mgc/$${base}.mgc`" ]; then \
				echo " Failed to extract MGC coefficients from $${raw}"; \
				rm -f mgc/$${base}.mgc; \
			fi; \
		fi; \
	done

	rm -f tmp

lf0:
	# Extracting log f0 sequence from raw audio
	mkdir -p lf0

	SAMPKHZ=`echo $(SAMPFREQ) | $(X2X) +af | $(SOPR) -m 0.001 | $(X2X) +fa`; \
	for raw in raw/$(DATASET)_$(SPEAKER)_*.raw; do \
		base=`basename $${raw} .raw`; \
		min=`$(X2X) +sf $${raw} | $(MINMAX) | $(X2X) +fa | head -n 1`; \
		max=`$(X2X) +sf $${raw} | $(MINMAX) | $(X2X) +fa | tail -n 1`; \
		if [ -s $${raw} -a $${min} -gt -32768 -a $${max} -lt 32767 ]; then \
			echo "Extracting f0 from $${raw}"; \
			$(X2X) +sf $${raw} | $(PITCH) -H $(UPPERF0) -L $(LOWERF0) -p $(FRAMESHIFT) -s $${SAMPKHZ} -o 2 > lf0/$${base}.lf0; \
			if [ -n "`$(NAN) lf0/$${base}.lf0`" ]; then \
				echo " Failed to extract log f0 from $${raw}"; \
				rm -f lf0/$${base}.lf0; \
			fi; \
		fi; \
	done

cmp: 
	# Composing training data files from mgc and lf0 files
	mkdir -p cmp

	for j in 1; do \
		MGCDIM=`expr $(MGCORDER) + 1`; \
		LF0DIM=1; \
		MGCWINDIM=`expr $(NMGCWIN) \* $${MGCDIM}`; \
		LF0WINDIM=`expr $(NLF0WIN) \* $${LF0DIM}`; \
		BYTEPERFRAME=`expr 4 \* \( $${MGCWINDIM} + $${LF0WINDIM} \)`; \
		for mgc in mgc/$(DATASET)_$(SPEAKER)_*.mgc; do \
			base=`basename $${mgc} .mgc`; \
			MGCWINS=""; \
			LF0WINS=""; \
			if [ -s $${mgc} -a -s lf0/$${base}.lf0 ]; then \
				i=1; \
				echo "Composing training data for $${base}"; \
				while [ $${i} -le $(NMGCWIN) ]; do \
					eval MGCWINS=\"$${MGCWINS} $(MGCWIN)$${i}\"; \
					i=`expr $${i} + 1`; \
				done; \
				$(PERL) scripts/window.pl $${MGCDIM} $${mgc} $${MGCWINS} > cmp/tmp.mgc; \
				i=1; \
				while [ $${i} -le $(NLF0WIN) ]; do \
					eval LF0WINS=\"$${LF0WINS} $(LF0WIN)$${i}\"; \
					i=`expr $${i} + 1`; \
				done; \
				$(PERL) scripts/window.pl $${LF0DIM} lf0/$${base}.lf0 $${LF0WINS} > cmp/tmp.lf0; \
				$(MERGE) +f -s 0 -l $${LF0WINDIM} -L $${MGCWINDIM} cmp/tmp.mgc < cmp/tmp.lf0 > cmp/tmp.cmp; \
				$(PERL) scripts/addhtkheader.pl $(SAMPFREQ) $(FRAMESHIFT) $${BYTEPERFRAME} 9 cmp/tmp.cmp > cmp/$${base}.cmp; \
			fi; \
		done; \
	done
	rm -f cmp/tmp*

label:
	# Extracting monophone and fullcontext labels from utterance files
	mkdir -p labels/mono
	mkdir -p labels/full

	for utt in utts/$(DATASET)_$(SPEAKER)_*.utt; do \
		base=`basename $${utt} .utt`; \
		if [ -s $${utt} ]; then \
			$(DUMPFEATS)    -eval     scripts/extra_feats.scm \
					-relation Segment \
					-feats    scripts/label.feats \
					-output   tmp \
					$${utt}; \
		fi; \
		$(AWK) -f scripts/label-full.awk tmp > labels/full/$${base}.lab; \
		$(AWK) -f scripts/label-mono.awk tmp > labels/mono/$${base}.lab; \
	done
	rm -f tmp

mlf:
	# Generating monophone and fullcontext Master Label Files (MLF)
	echo "#!MLF!#" > labels/mono.mlf
	echo "\"*/$(DATASET)_$(SPEAKER)_*.lab\" -> \"/home/nlphacker/work/OpenSpeech/voices/HTS-AWB/data/labels/mono\"" >> labels/mono.mlf

	echo "#!MLF!#" > labels/full.mlf
	echo "\"*/$(DATASET)_$(SPEAKER)_*.lab\" -> \"/home/nlphacker/work/OpenSpeech/voices/HTS-AWB/data/labels/full\"" >> labels/full.mlf

list:
	# Generating a fullcontext model list file
	mkdir -p lists

	rm -f tmp
	for lab in labels/full/$(DATASET)_$(SPEAKER)_*.lab; do \
		if [ -s $${lab} -a -s labels/mono/`basename $${lab}` -a -s cmp/`basename $${lab} .lab`.cmp ]; then \
			sed -e "s/.* //g" $${lab} >> tmp; \
		fi \
	done
	sort -u tmp > lists/full.list
	rm -f tmp

	# Generating a fullcontext model list file which includes unseen models
	rm -f tmp
	cat lists/full.list > tmp
	for lab in labels/gen/*.lab; do \
		sed -e "s/.* //g" $${lab} >> tmp; \
	done
	sort -u tmp > lists/full_all.list
	rm -f tmp

	# Generating a monophone list file
	rm -f tmp
	for lab in labels/mono/$(DATASET)_$(SPEAKER)_*.lab; do \
		if [ -s $${lab} -a -s labels/full/`basename $${lab}` -a -s cmp/`basename $${lab} .lab`.cmp ]; then \
			sed -e "s/.* //g" $${lab} >> tmp; \
		fi \
	done
	sort -u tmp > lists/mono.list
	rm -f tmp

scp:
	# Generating a trainig data script
	mkdir -p scp

	rm -f scp/train.scp
	for cmp in /home/nlphacker/work/OpenSpeech/voices/HTS-AWB/data/cmp/$(DATASET)_$(SPEAKER)_*.cmp; do \
		if [ -s $${cmp} -a -s labels/mono/`basename $${cmp} .cmp`.lab -a -s labels/full/`basename $${cmp} .cmp`.lab ]; then \
			echo $${cmp} >> scp/train.scp; \
		fi \
	done

	# Generating a generation label script
	rm -f scp/gen.scp
	for lab in /home/nlphacker/work/OpenSpeech/voices/HTS-AWB/data/labels/gen/*.lab; do \
		echo $${lab} >> scp/gen.scp; \
	done


clean: clean-mgc clean-lf0 clean-cmp clean-label clean-mlf clean-list clean-scp

clean-mgc:
	rm -rf mgc

clean-lf0:
	rm -rf lf0

clean-cmp:
	rm -rf cmp

clean-label:
	rm -rf labels/mono
	rm -rf labels/full

clean-mlf:
	rm -f labels/*.mlf

clean-list:
	rm -rf lists

clean-scp:
	rm -rf scp

distclean: clean
	rm -f Makefile

.PHONY: all analysis labels mgc lf0 cmp label mlf list scp clean
