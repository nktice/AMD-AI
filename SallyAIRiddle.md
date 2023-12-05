# AI test riddle Sally - and 'Simple Sally'

# Question 1:
I saw this question on a post on reddit and thought it was worth trying...
`"Sally (a girl) has 3 brothers. Each brother has 2 sisters. How many sisters does Sally have?"`

These settings appear important to get things to work so they are default:
exllama [loader] : max_seq_len 4096, compress_pos_emb 4

TheBloke_13B-Legerdemain-L2-GPTQ - [fail]
[deluded hallucinations.]

TheBloke_30B-Epsilon-GPTQ - [fail]
[deluded hallucinations]

TheBloke_Airoboros-7B-GPT4-1-4-SuperHOT-8K-GPTQ - [fail]
"1.5", "2" ...

TheBloke_Airoboros-c34B-3.1.2-GPTQ - [fail] 
`"Sally has 6 sisters,[...]"`

TheBloke_airoboros-33B-GPT4-2.0-GPTQ - [fail]
`"None, because she herself is a girl."`

TheBloke_airoboros-33B-GPT4-m2.0-GPTQ - [fail]
[ 4, 2, ... ]

TheBloke_Airolima-Chronos-Grad-L2-13B-GPTQ - [faiil]
[ random answers, short but deluded. ]

TheBloke_Airolima-Chronos-Grad-L2-13B-GPTQ - [ pass ]
`"Sally has 1 sister."`
[seed 1( correct 1/3),  5(2/6 correct) ... rarely passing, and unstable. ] 

TheBloke_Augmental-13B-GPTQ - [ fail ] 
`"None [...]"` 

TheBloke_Cat-13B-0.5-GPTQ - [ fail ] 
`"She has no brothers![...]"` [deluded] 

TheBloke_Chronolima-Airo-Grad-L2-13B-GPTQ - [fail]
[ 2... ]

TheBloke_CodeFuse-CodeLlama-34B-GPTQ - [fail] 
`"Sally has 6 sisters."`

TheBloke_CodeLlama-34B-Instruct-GPTQ - [ fail ]
[ insists there's 6 sisters. ] 

TheBloke_CodeUp-Alpha-13B-HF-GPTQ - [fail]
[ mostly answers 6, sometimes doesn't know ] 

TheBloke_CodeUp-Llama-2-13B-Chat-HF-GPTQ - [fail]
[ 6 ]

TheBloke/dolphin-2.1-mistral-7B-GPTQ - [fail]
`"[...] She has 2 sisters.[...]"`

TheBloke_GodziLLa-30B-GPTQ - [fail]
[ various answers, does sometiems say it doesn't understand or needs more info. ] 

TheBloke_gorilla-7B-GPTQ - [fail]
[ Of a dozen runs, no answer was correct. ] 

TheBloke_gpt4-alpaca-lora-30B-GPTQ-4bit-128g - [fail]

TheBloke_guanaco-33B-GPTQ - [mixed]
[ correct 1/3 ?!? ] 

TheBloke_Guanaco-33B-SuperHOT-8K-GPTQ - [fail]
[ answers vary, but are incorrect... often 4 or 6. ] 

TheBloke_hippogriff-30b-chat-GPTQ - [fail]
`"... Sally has 6 sisters."`

TheBloke_Llama-2-13B-GPTQ - [fail]
[ deluded ]

TheBloke_llama2-22B-daydreamer-v2-GPTQ - [fail]
[ deluded, verbose ] 

TheBloke_Llama2-22B-Daydreamer-v3-GPTQ - [fail]
[ wrong, verbose... ]

TheBloke_llama2_7b_chat_uncensored-GPTQ - [fail]
[ 2 ]

TheBloke_llava-v1.5-13B-GPTQ - [fail]
`"0"`

TheBloke_llemma_34b-GPTQ - [fail]
[ gives a range of answers... ]

TheBloke_LongChat-7B-GPTQ - [fail]
[ 0 ]

TheBloke_LosslessMegaCoder-Llama2-13B-Mini-GPTQ - [fail]
[ tends to get caught in a loop.  deluded. ] 

TheBloke_LosslessMegaCoder-Llama2-7B-Mini-GPTQ - [fail]
[ of a dozen tries, varies between 6 and 7, sometimes other numbers. ] 

TheBloke_Luna-AI-Llama2-Uncensored-GPTQ - [fail]
[ numbers differ / wrong ]

TheBloke_MAmmoTH-Coder-34B-GPTQ - [fail]
`"Sally has 3 * 2 = <<3*2=6>>6 sisters.6"`

TheBloke/Mistral-11B-CC-Air-RP-GPTQ - [fail]

TheBloke_MythoMax-L2-33B-GPTQ - [fail]
`"None. She only one sister named Bethany."`

TheBloke_NeuralHermes-2.5-Mistral-7B-GPTQ - [partial] 
this is pretty close... 
`" She has 2 sisters.
[...] reasoning: The question states that Sally is a girl and she has 3 brothers, so we know there are at least two girls in the family - Sally and one other sister. Since each of her three brothers also have 2 sisters, it means they all share these same two sisters. Therefore, Sally must be one of those two sisters."`

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

TheBloke_Orca-2-13B-GPTQ - [fail] 

TheBloke_Phind-CodeLlama-34B-v2-GPTQ - [fail]
`"Sally has three brothers, and each of her brothers has two sisters. This means that there are six sisters in total who are not Sally. Therefore, Sally herself must be one of these seven sisters. So, Sally has a total of eight sisters including herself."`  

TheBloke_Pygmalion-2-13B-GPTQ - [mixed] 
`"Let me think..."`
It's not the answer, but it's not pretending that it has the answer right when it doesn't.  

TheBloke_Pygmalion-2-13B-SuperCOT-GPTQ - [fail, mixed...]
Insisted Sally had 6 sisters, but when I followed up by asking, "What if the brothers' two sisters are the same 2 people, so there's only 2 girls in that generation?", it corrected itself and said, "Sally has 1 sister."

TheBloke_robin-33B-v2-GPTQ - [fail]
[ gives wrong answer, and then goes on and on in delusion... ] 

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

TheBloke_SuperPlatty-30B-GPTQ - [fail] 

TheBloke_Synthia-34B-v1.2-GPTQ - [fail]
`"Sally has 6 sisters."`
 
TheBloke/speechless-codellama-34b-v2.0-GPTQ - [fail]
`"Sally has 6 sisters."`

TheBloke/Tulu-30B-SuperHOT-8K-GPTQ - [fail]
`"Sally has 1 + 3 = 4 siblings in total. So she has 4 * 2 = 8 sisters - in - law."`

TheBloke_upstage-llama-30b-instruct-2048-GPTQ - [fail]

TheBloke_vicuna-33B-GPTQ 
[new version 2023-09-26 ] [ mixed ] - Sometimes answers 1, sometimes 0, or other answers. 

[ 2023-08-20 ] - [mixed]
[ Consistently says it can't answer... which is better than insisting on a wrong answer. ] 
`"Based on the information provided, Sally has 3 brothers, but it is not possible to determine how many sisters she has as there is no information about her own siblings other than her brothers."`

TheBloke_WizardLM-1.0-Uncensored-CodeLlama-34B-GPTQ - [fail]
"Sally has 6 sisters."

TheBloke_Wizard-Vicuna-7B-Uncensored-SuperHOT-8K-GPTQ - [<mixed]
1:"2"
2:"4"
3:`"[...] Sally herself also has at least one sister. "`
[8k/8] [fail]

TheBloke_Wizard-Vicuna-30B-Uncensored-GPTQ - [fail]
Insists : `"Sally doesn't have any sisters herself, but her three brothers each have two sisters so in total she has six siblings."`

TheBloke_Wizard-Vicuna-30B-Uncensored-GPTQ - [fail]
[ no sisters ]

TheBloke_WizardLM-1.0-Uncensored-Llama2-13B-GPTQ - [fail]
[ Consistantly says 6 sisters in different ways... ] 
`"Sally has 6 sisters in total."`

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

TheBloke_Airoboros-c34B-3.1.2-GPTQ - [ pass ] 
`"Sally has 1 sister.[...]"`

TheBloke_airoboros-33B-GPT4-2.0-GPTQ - [ pass ]
`"If Sally herself is a girl, then she has at least one sister - another girl in her family."`

TheBloke_airoboros-33B-GPT4-m2.0-GPTQ - [ pass ]
`"Sally has 1 sister."`

TheBloke_Airolima-Chronos-Grad-L2-13B-GPTQ - [ pass ]
`"Sally has one sister because there are only two girls in total out of five children, so she has one sister."`
[ seed 1,2, 9(flip flops, mostly right),10 = pass ] 

TheBloke_Augmental-13B-GPTQ - [ pass ] 
`"1"`

TheBloke_Cat-13B-0.5-GPTQ - [ fail ] 
`"[...] Sally has two sisters."`

TheBloke_Chronoboros-33B-GPTQ - [ pass ]
`"Sally has 1 sister."`

TheBloke_Chronolima-Airo-Grad-L2-13B-GPTQ - [ pass ]
`"Sally has one sister because she is one of two girls in her family."`

TheBloke_CodeFuse-CodeLlama-34B-GPTQ - [ pass ] 
`"Sally has 1 sister."`

TheBloke_CodeLlama-34B-Instruct-GPTQ - [ pass ]
`"Sally has 1 sister."`

TheBloke_CodeUp-Alpha-13B-HF-GPTQ - [fail]
[ out of 12 attempts, 10 were '2', and 2 were '3' ] 

TheBloke_CodeUp-Llama-2-13B-Chat-HF-GPTQ - [ fail. ]
`"Hi there! Based on the information provided, Sally has 2 sisters."`

TheBloke/dolphin-2.1-mistral-7B-GPTQ - [pass]
`"[...] we can only conclude that she has at least one sister"`

TheBloke_GodziLLa-30B-GPTQ - [ fail ]
[ gets close to answering, but the breaks it with confusion about gender or other issues. ] 

TheBloke_gorilla-7B-GPTQ - [ fail ] 
[ Of a dozen tries, there was no correct answer. ] 

TheBloke_gpt4-alpaca-lora-30B-GPTQ-4bit-128g - [ fail ]
[ correct once out of a dozen tries... ] 

TheBloke_guanaco-33B-GPTQ - [pass]
`"Sally has one sister"` [consistently!] 

TheBloke_Guanaco-33B-SuperHOT-8K-GPTQ - [mixed]
[ answers 1 about half the time out of a dozen random tests. ] 

TheBloke_hippogriff-30b-chat-GPTQ - [pass] 
`"Sally has 1 sister because she is one of the 2 girls in her family."`

TheBloke_llama2-22B-daydreamer-v2-GPTQ - [fail]
[ From several attempts: Deluded, verbose, repititious. ] 

TheBloke_Llama2-22B-Daydreamer-v3-GPTQ - [fail]
[runs on and on... ]

TheBloke_Llama-2-13B-GPTQ - [ 1/2 ? ]
`"The answer is 1. [...]"` - This was 2nd try... first was wrong.

TheBloke_llama2_7b_chat_uncensored-GPTQ - [2/3 mixed pass ... ]
`"none", "1", "Sally has 1 sister - her brother's name is John."`

TheBloke_llava-v1.5-13B-GPTQ - [fail]
`"0"`

TheBloke_llemma_34b-GPTQ - [pass / mixed ] 
Answer does include :
`"[...] The correct answer to this question is 1. Since there are three boys and two girls in the family, we can conclude that Sally must be one of the two girls. Therefore, she only has one sister. [...]"`
Alas it has a bunch more in the answer that is not so correct. 

TheBloke_LongChat-7B-GPTQ -  [ fail ]
Out of several tries there were different errors and wrong answers.

TheBloke_LosslessMegaCoder-Llama2-13B-Mini-GPTQ - [mixed - pass]
[ does give a correct answer, but tends to get caught in a loop until it runs out of tokens ]  

TheBloke_LosslessMegaCoder-Llama2-7B-Mini-GPTQ - [mixes] 
[ Does sometimes answer 1... 
[ seed of 0-4 pass, 5 fail, ...? ]

TheBloke_Luna-AI-Llama2-Uncensored-GPTQ - [ pass ]
`"Sally has one sister."`

TheBloke_MAmmoTH-Coder-34B-GPTQ - [ pass ] 
`"Sally has 4 siblings, 3 brothers and 1 sister"`

TheBloke/Mistral-11B-CC-Air-RP-GPTQ - [fail]
`"There are 4 girls in this family including Sally, so she has 3 sisters."`

TheBloke_MythoMax-L2-33B-GPTQ - [fail]
With minimal temp, gave a few different random odd answers.

TheBloke_NeuralHermes-2.5-Mistral-7B-GPTQ - [pass]
`" She has 1 sister."`

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

TheBloke_Orca-2-13B-GPTQ - [ pass ] 
`"Sally has only one sister because she is the other girl in the family."`

TheBloke_Phind-CodeLlama-34B-v2-GPTQ - [pass]
`"Sally has 1 sister."`

TheBloke_Pygmalion-2-13B-GPTQ - [mixed] 
`"Let me think..."`
It's not the right answer, but it's not a wrong answer.  

TheBloke_Pygmalion-2-13B-SuperCOT-GPTQ - [fail]
insists : `"To answer this question, we should know that: Sally has 1 brother and 2 sisters."`

TheBloke/robin-33B-v2-GPTQ - [mixed]
[ Starts out with correct answer but then goes on and on rambling incoherently.  ]

TheBloke_Samantha-13B-SuperHOT-8K-GPTQ - [ mixes / pass ]
`"Based on your description, there would be three siblings total two brothers and one sister."` 
[ Sort of wrong, sort of right... ]

TheBloke_Samantha-33B-SuperHOT-8K-GPTQ -  [ pass ]
`"Sally has 1 biological sister, as she is one of the 5 children in the family, with 3 boys and 2 girls."`

TheBloke_Stable-Platypus2-13B-GPTQ - [ mixed / pass ]
`"Sally has 1 sister since she is a girl among the other 4 children who are also girls."` 
[ sort of right, sort of wrong... ]

TheBloke_SuperPlatty-30B-GPTQ - [fail]
[false answers / confused. ] 

TheBloke_Synthia-34B-v1.2-GPTQ - [pass]
`"Sally has 1 sister."`

TheBloke/speechless-codellama-34b-v2.0-GPTQ - [pass]
`"Sally has 1 sister."`

TheBloke/Tulu-30B-SuperHOT-8K-GPTQ - [pass]
`"Sally has 1 sister as she is one of 5 children from the same two parents who had 3 boys and 2 girls."`

TheBloke_upstage-llama-30b-instruct-2048-GPTQ -  [ pass ]
`"Since there are only two girls in the family, including Sally herself, Sally has one sister."`

TheBloke_vicuna-33B-GPTQ - [pass]
[2023-09-29] - [pass] `"Sally has 1 sister."`
[2023-08-20 ] - [ fairly consistent ] 
`"Based on the information provided, Sally has three siblings - two brothers and one sister."`

TheBloke_Wizard-Vicuna-7B-Uncensored-SuperHOT-8K-GPTQ - [ fail ]
`"Sally has 1 brother and 4 sisters."`

TheBloke_Wizard-Vicuna-30B-Uncensored-GPTQ - [pass]
`"If Sally is a girl, she could be either the oldest or youngest child in the family with three older brothers and one younger brother or one of two older siblings who are also girls. Therefore, she has at least one sister."`


TheBloke_Wizard-Vicuna-30B-Uncensored-GPTQ - [ pass ]
`"If Sally is a girl, she has only one sister who is also a girl, since there are only two girls in the family out of five children."`

TheBloke_WizardLM-1.0-Uncensored-CodeLlama-34B-GPTQ - [mixed / fail] 
1 out of 3 random gave the correct answer (`"Sally has 1 sister."`) 
I tried setting seeds of 0 through 10, and they all said `"Sally has 6 sisters."`


TheBloke_WizardLM-1.0-Uncensored-Llama2-13B-GPTQ - [fail]
[ Consistent but off by 1 ] 
`"Sally has 2 sisters"`

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
 [ seed 0-10,...? pass ]

TheBloke_WizardMath-13B-V1.0-GPTQ - [ fail ]
`"The answer is: 2."`

###### - Initial tests performed on 2023-08-11 - see changelog for updates.



