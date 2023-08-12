# AI test riddle Sally - and 'Simple Sally'

# Question 1:
I saw this question on a post on reddit and thought it was worth trying...
`"Sally (a girl) has 3 brothers. Each brother has 2 sisters. How many sisters does sally have?"`

These settings appear important to get things to work so they are default:
exllama [loader] : max_seq_len 4096, compress_pos_emb 4

TheBloke_13B-Legerdemain-L2-GPTQ - [fail]
[deluded hallucinations.]

TheBloke_30B-Epsilon-GPTQ - [fail]
[deluded hallucinations]

TheBloke_Airoboros-7B-GPT4-1-4-SuperHOT-8K-GPTQ - [fail]
"1.5", "2" ...

TheBloke_airoboros-33B-GPT4-2.0-GPTQ - [fail]
`"None, because she herself is a girl."`

TheBloke_airoboros-33B-GPT4-m2.0-GPTQ - [fail]
[ 4, 2, ... ]

TheBloke_Airolima-Chronos-Grad-L2-13B-GPTQ - [faiil]
[ random answers, short but deluded. ]

TheBloke_Airolima-Chronos-Grad-L2-13B-GPTQ - [ pass ]
`"Sally has 1 sister."`
[seed 1( correct 1/3),  5(2/6 correct) ... rarely passing, and unstable. ] 

TheBloke_Chronolima-Airo-Grad-L2-13B-GPTQ - [fail]
[ 2... ]

TheBloke_CodeUp-Llama-2-13B-Chat-HF-GPTQ - [fail]
[ 6 ]

TheBloke_Llama-2-13B-GPTQ - [fail]
[ deluded ]

TheBloke_llama2_7b_chat_uncensored-GPTQ - [fail]
[ 2 ]

TheBloke_LongChat-7B-GPTQ - [fail]
[ 0 ]

TheBloke_Luna-AI-Llama2-Uncensored-GPTQ - [fail]
[ numbers differ / wrong ]

TheBloke_OpenAssistant-Llama2-13B-Orca-8K-3319-GPTQ  - [mixed]
3rd answer - `" Sally has one sister. "`
[ trying at 8k/8 did not improve results - they became worse... ]

TheBloke_orca_mini_v2_7B-GPTQ - [mixed]
1:`"Sally has half as many siblings as her brothers, since each of her brothers has two sisters and
 she has none. Therefore, she has one sister."`
2: 2 sisters...
3: deluded...

TheBloke_orca_mini_v2_13b-GPTQ - [mixed]
2nd answer... `"[...] Therefore, so Sally has one sister. "`
[ answers mixed ]
[ loading in 8K/8 makes answers stable, but wrong. ]
3: `"[...] Therefore, Sally has one sister."`


TheBloke_orca_mini_v3_7B-GPTQ - [fail]

TheBloke_orca_mini_v3_13B-GPTQ - [crash] [fail]

TheBloke_Samantha-13B-SuperHOT-8K-GPTQ - [fail]
[4k/4 does produce output but it's incorrect ] 
[8k/8 also fails ]

TheBloke_Samantha-33B-SuperHOT-8K-GPTQ - [fail]
[4k/4 does produce output but it's incorrect ] 
[8k/8 crash ]
[8k/8 out of memory. ]
[8k/8 gpu_split 12,16 - loads but errors... ]
[8k/8 gpu_split 10,16 - no errors, but produces garbage. ]

TheBloke_Stable-Platypus2-13B-GPTQ - [fail]

TheBloke_upstage-llama-30b-instruct-2048-GPTQ - [fail]

TheBloke_Wizard-Vicuna-7B-Uncensored-SuperHOT-8K-GPTQ - [<mixed]
1:"2"
2:"4"
3:`"[...] Sally herself also has at least one sister. "`
[8k/8] [fail]

TheBloke_Wizard-Vicuna-30B-Uncensored-GPTQ - [fail]
[ no sisters ]

TheBloke_WizardLM-13B-V1.0-Uncensored-GPTQ - [fail]


TheBloke_WizardLM-30B-Uncensored-GPTQ - [pass/mixed]
1 :`"Sally has 1 sister."`
2 : 0
3 : 1
4 : 6
5 : 1
[ 4k,4, 2048 tokens, Seeds of[ 4-10 ...? ], yeid `Sally has 1 sister.` ] 

TheBloke_WizardLM-33B-V1-0-Uncensored-SuperHOT-8K-GPTQ - [fail]
[deluded.  answers differ... ]
[8k/8 crash... out of memory. ]
[ 12, 12, 12... ]

TheBloke_WizardLM-Uncensored-SuperCOT-StoryTelling-30B-GPTQ - [fail]
[ confused ]

TheBloke_WizardLM-Uncensored-SuperCOT-StoryTelling-30B-SuperHOT-8K-GPTQ -[fail]

TheBloke_WizardMath-13B-V1.0-GPTQ - [fail]


-----------------


# Quesiton 2:
As some systems had trouble with the previous question, I simplified it...
[ perhaps we could call this, 'Simple Sally' compared to the originial 'Sally' question? ] 

`"Sally ( a girl ) is one of 5 children from the same two parents. The parents had 3 boys, and 2 girls. How many sisters does Sally have?" `

These settings appear important to get things to work so they are default:
exllama [loader] : max_seq_len 4096, compress_pos_emb 4

TheBloke_13B-Legerdemain-L2-GPTQ - [ fail ]
`"Two. She has an older sister named Laura and a younger sister named Jenny."`

TheBloke_30B-Epsilon-GPTQ  -  [ pass ]
`"Sally has 1 sister since she herself is also a girl, making it 4 siblings in total with 3 brothers and 1 other sister."`

TheBloke_Airoboros-7B-GPT4-1-4-SuperHOT-8K-GPTQ - [ fail ]
`"Sally has 2 sisters. [...]"`

TheBloke_airoboros-33B-GPT4-2.0-GPTQ - [ pass ]
`"If Sally herself is a girl, then she has at least one sister - another girl in her family."`

TheBloke_airoboros-33B-GPT4-m2.0-GPTQ - [ pass ]
`"Sally has 1 sister."`

TheBloke_Airolima-Chronos-Grad-L2-13B-GPTQ - [ pass ]
`"Sally has one sister because there are only two girls in total out of five children, so she has one sister."`
[ seed 1,2, 9(flip flops, mostly right),10 = pass ] 

TheBloke_Chronoboros-33B-GPTQ - [ pass ]
`"Sally has 1 sister."`

TheBloke_Chronolima-Airo-Grad-L2-13B-GPTQ - [ pass ]
`"Sally has one sister because she is one of two girls in her family."`

TheBloke_CodeUp-Llama-2-13B-Chat-HF-GPTQ - [ fail. ]
`"Hi there! Based on the information provided, Sally has 2 sisters."`

TheBloke_Llama-2-13B-GPTQ - [ 1/2 ? ]
`"The answer is 1. [...]"` - This was 2nd try... first was wrong.

TheBloke_llama2_7b_chat_uncensored-GPTQ - [2/3 mixed pass ... ]
`"none", "1", "Sally has 1 sister - her brother's name is John."`

TheBloke_LongChat-7B-GPTQ -  [ fail ]
Out of several tries there were different errors and wrong answers.

TheBloke_Luna-AI-Llama2-Uncensored-GPTQ - [ pass ]
`"Sally has one sister."`

TheBloke_OpenAssistant-Llama2-13B-Orca-8K-3319-GPTQ - [ 1/4 ? ]
It took 4 tries to get - `"Sally has 1 sister."`

TheBloke_orca_mini_v2_13b-GPTQ -  [ pass ]
`"Sally has 1 sister."`

TheBloke_orca_mini_v2_7B-GPTQ -  [ .5 / 4 ? fail  ]
out of 4 answers, the 2nd was convoluted but ended correctly.  [ others wrong.]

TheBloke_orca_mini_v3_7B-GPTQ - [ pass ]
`"Sally has 1 sister."`

TheBloke_orca_mini_v3_13B-GPTQ - [ fail ]
`"Sally has 2 sisters, as she is one of the 2 girls among the 5 children."`

TheBloke_Samantha-13B-SuperHOT-8K-GPTQ - [ mixes / pass ]
`"Based on your description, there would be three siblings total two brothers and one sister."` 
[ Sort of wrong, sort of right... ]

TheBloke_Samantha-33B-SuperHOT-8K-GPTQ -  [ pass ]
`"Sally has 1 biological sister, as she is one of the 5 children in the family, with 3 boys and 2 girls."`

TheBloke_Stable-Platypus2-13B-GPTQ - [ mixed / pass ]
`"Sally has 1 sister since she is a girl among the other 4 children who are also girls."` 
[ sort of right, sort of wrong... ]

TheBloke_upstage-llama-30b-instruct-2048-GPTQ -  [ pass ]
`"Since there are only two girls in the family, including Sally herself, Sally has one sister."`

TheBloke_Wizard-Vicuna-7B-Uncensored-SuperHOT-8K-GPTQ - [ fail ]
`"Sally has 1 brother and 4 sisters."`

TheBloke_Wizard-Vicuna-30B-Uncensored-GPTQ - [ pass ]
`"If Sally is a girl, she has only one sister who is also a girl, since there are only two girls in the family out of five children."`

TheBloke_WizardLM-13B-V1.0-Uncensored-GPTQ - [ mixed - fail ]
`" [...] So, Sally has at least one sister out of the remaining two girls. "`

TheBloke_WizardLM-30B-Uncensored-GPTQ - [ mixed .5 fail ]
1st answer said no sisters. 2nd answer ended `"[...] Therefore, Sally has one sister."`
[ 4k,4, 2048 tokens, Seed of 1,2,4-7,10 ...? passes " ] 

TheBloke_WizardLM-33B-V1-0-Uncensored-SuperHOT-8K-GPTQ - [ mixed pass ]
2nd answer ends `"[...] Sally has only one sister as she herself is also a girl."`


TheBloke_WizardLM-Uncensored-SuperCOT-StoryTelling-30B-GPTQ - [ pass ]
`"Sally has one sister. Since there were only two girls among the five children and she was one of them, she had no other sisters in this particular sibling group."`

TheBloke_WizardLM-Uncensored-SuperCOT-StoryTelling-30B-SuperHOT-8K-GPTQ -[pass]
`"Sally has 1 sister, as there were only 2 girls among the 5 children from the same two parents who
 had 3 boys. So, all the other females are either Sally or her mother, and there's just one 
 additional female - the mother herself."`

TheBloke_WizardMath-13B-V1.0-GPTQ - [ fail ]
`"The answer is: 2."`

###### - Tests performed on 2023-08-11



