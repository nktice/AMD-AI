# AI test riddle Sally - and 'Simple Sally'
2023-12-13- the Mixtral models (GGUF) have been loaded with llama.cpp configured following the guide here : https://github.com/nktice/AMD-AI/blob/main/Mixtral.md


# Question 1:
I saw this question on a post on reddit and thought it was worth trying...
`"Sally (a girl) has 3 brothers. Each brother has 2 sisters. How many sisters does Sally have?"`

These settings appear important to get things to work so they are default:
exllama [loader] : max_seq_len 4096, compress_pos_emb 4

cc_Dolphin3.0-Llama3.1-8B - [ Fail ] 
"Since Sally has 3 brothers, and each brother has 2 sisters, we can calculate the total number of sisters as follows: Each brother has 2 sisters, so the 3 brothers together have 3 x 2 = 6 sisters. However, this includes all sisters, not just Sally's sisters. Since Sally herself is one of the sisters (as she has brothers), we must subtract her from the total count to find her direct sisters. Therefore, Sally has 6 - 1 = 5 sisters."

deepseekai_DeepSeek-R1-Distill-Llama-8B - [ pass ] 
"[...] Sally has 1 sister."

deepseekai_DeepSeek-R1-Distill-Qwen-14B - [ fail ] 
"Sally has 2 sisters." 

mistralai_Ministral-8B-Instruct-2410 - [ fail ] 
"Let's break this down. If each of Sally's brothers has 2 sisters, that means they share the same sisters. Since Sally is one of the sisters, she has 2 brothers who see her as one of their sisters. Therefore, Sally has 2 sisters."

Qwen-Qwen2.5-14B-Instruct-1M - [ pass ] 
"Let's break it down the information given: Sally has 3 brothers, and each of those brothers has 2 sisters, who would include Sally herself. So, the 2 sisters are the same for all the brothers. Therefore, Sally has 2 sisters in total, including herself, making her one of them."

Qwen-Qwen2.5-Coder-32B-Instruct - [ pass ] 
"Let's break down the problem:  Sally has 3 brothers. Each of these 3 brothers has 2 sisters. Since Sally is one of the sisters, the other sister must be the only one shared sister among all the brothers. Therefore, Sally and her sister are the two sisters.  So, Sally has 1 sister."


TheBloke_Amber-GPTQ - [2x+fail]
gives bogus URL... extra double fail for not even trying and lying about it. 

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

TheBloke_dolphin-2.5-mixtral-8x7b-GGUF - [pass]
`"Factual answer: 1
Explanation: The factual answer is that Sally only has one sister, herself. She doesn't count as a "sister" to her own self in this context. So the number of siblings she has (3 brothers and 1 sister) equals the total number of sisters each brother has combined (2)."`

TheBloke_dolphin-2.6-mixtral-8x7b-GGUF - [pass]
`"Based on the information provided, each of Sally's 3 brothers has 2 sisters in total. Since all three brothers share the same set of sisters, that means there are only two other girls besides Sally. Therefore, Sally has one sister."` - perfect. 

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

TheBloke_mistral-7B-dpo-v5-GPTQ - [fail]
`"Since the information given only tells us that each of Sally's brothers has two sisters, we cannot determine how many sisters Sally herself has. We need more information to solve this problem."`

TheBloke_Mistral-7B-Instruct-v0.2-GPTQ - [close...]
`"[...] So when we say "Sally's brothers each have 2 sisters," we mean that in total there are 2 sisters in the family [...]"`

TheBloke/Mistral-7B-Instruct-v0.2-DARE-GPTQ - [fail] 
 answer varies, out of several attempts it answered 3, and 6... 

TheBloke_mistral_7b_norobots-GPTQ - [pass]
`" Sally has 1 sister. "`

TheBloke_Mixtral-8x7B-Instruct-v0.1-GGUF - [fail]
starts out great, and then falls on its face... 
`" Sally has one sister, herself. The other two girls are her brothers’ sisters but not hers because they aren't related by blood to each other or their parents. [...]"` 

TheBloke_Mixtral-8x7B-v0.1-GGUF - [pass]
`"1 sister, because she is one of the two girls in her family and each boy only has one other sister besides herself! [...] "`

TheBloke_Mixtral-8x7B-MoE-RP-Story-GGUF - [fail]
 `"Factual answer: 0
Correct answer: 3"` 

TheBloke_Mixtral-SlimOrca-8x7B-GGUF - [fail]
`"Sally has no sisters because she is a girl, not a boy. She cannot be one of her own brothers' sisters."`

TheBloke_mixtralnt-4x7b-test-GGUF - [fail]
`" 3 brothers x 2 sisters = 6 sisters "`

TheBloke_MythoMax-L2-33B-GPTQ - [fail]
`"None. She only one sister named Bethany."`

TheBloke_Nethena-20B-Glued-GPTQ - [fail]
says there's 4 sisters. 

TheBloke_NeuralHermes-2.5-Mistral-7B-GPTQ - [partial] 
this is pretty close... 
`" She has 2 sisters.
[...] reasoning: The question states that Sally is a girl and she has 3 brothers, so we know there are at least two girls in the family - Sally and one other sister. Since each of her three brothers also have 2 sisters, it means they all share these same two sisters. Therefore, Sally must be one of those two sisters."`

TheBloke_NeuralOrca-7B-v1-GPTQ - [partial]
`"[...] 
Explanation: The question states that Sally has 3 brothers, and each of her brothers has 2 sisters. Since Sally is one of the two sisters for each brother, she must also be a sister to herself. Therefore, Sally has only 1 more sister besides herself, making it a total of 2 sisters."`

TheBloke_neural-chat-7B-v3-3-GPTQ - [ fail ] 
hasn't figured out that the sisters are shared - assuming there's 6. 

TheBloke_OpenAssistant-Llama2-13B-Orca-8K-3319-GPTQ  - [mixed]
3rd answer - `" Sally has one sister. "`
[ trying at 8k/8 did not improve results - they became worse... ]

TheBloke_openbuddy-mixtral-8x7b-v15.1-GGUF - [pass]
`"Factual answer: None, because there is no information given about how many siblings each of her brothers' two sisters are related to or not. The problem as stated cannot be solved with the available data. However, if we assume that all three brothers and their respective 2 sisters form a single family unit (which may or may not be true), then Sally would have at least one sister who is also part of her immediate family - but this assumption has no basis in factual information provided by the question itself."`

TheBloke_openchat-3.5-1210-GPTQ - [pass]
when asked "Answer this question exactly."
answers : "Sally has 1 sister."

TheBloke_OpenHermes-2.5-Mistral-7B-GPTQ - [pass] 
`"She has 1 sister."`

TheBloke_OpenHermes-2.5-neural-chat-v3-3-Slerp-GPTQ - [fail]
thinks there's 6 sisters... answers 5. 

TheBloke_OpenZephyrChat-GPTQ - [pass]
When asked to explain answer, does give the correct answer, but does prattle on afterwards. 
`"Sally has 1 sister."`


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

TheBloke_Samantha-33B-SuperHOT-8K-GPTQ - [fail]

TheBloke_Stable-Platypus2-13B-GPTQ - [fail]

TheBloke_SuperPlatty-30B-GPTQ - [fail] 

TheBloke_Synthia-34B-v1.2-GPTQ - [fail]
`"Sally has 6 sisters."`

TheBloke_Synthia-MoE-v3-Mixtral-8x7B-GGUF - [fail]
`"4 sisters (excluding herself)"`
 
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


cc_Dolphin3.0-Llama3.1-8B - [ pass ] 
"Since Sally is one of the two girls and there are five children in total, and we know there are three boys, that means there must be two girls. Since Sally is one of those girls, she would have one sister."

deepseekai_DeepSeek-R1-Distill-Llama-8B - [ pass ] 
"[...]Sally has one sister."

deepseekai_DeepSeek-R1-Distill-Qwen-14B  - [fail] 
"Sally has two sisters."

mistralai_Ministral-8B-Instruct-2410 - [ pass ] 
"Sally has 1 sister. Since Sally is one of the girls and the parents have two girls, she has another sister."

Qwen-Qwen2.5-14B-Instruct-1M - [ pass ] 
"Sally has 1 sister. Since there are 5 children in total—3 boys and 2 girls—and Sally is one of them, that leaves 1 other girl who is her sister."

Qwen-Qwen2.5-Coder-32B-Instruct - [ pass ] 
"Sally has one sister, since she is one of the two girls among the five children."


TheBloke_Amber-GPTQ - [pass]
`"She has 1 Sister"`

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

TheBloke_dolphin-2.5-mixtral-8x7b-GGUF - [pass]
`"Sally has 1 sister."`

TheBloke_dolphin-2.6-mixtral-8x7b-GGUF - [pass]
1...

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

TheBloke_mistral-7B-dpo-v5-GPTQ - [pass]
`"Sally has 1 sister."`

TheBloke_Mistral-7B-Instruct-v0.2-GPTQ - [pass]
`"Sally has 1 sister."`

TheBloke_Mistral-7B-Instruct-v0.2-DARE-GPTQ - [pass]
`"Sally has 1 sister."`

TheBloke_mistral_7b_norobots-GPTQ - [fail]
`"None. There are no sisters for Sally because she is not a sister to any of her brothers or sisters. She is only a daughter."`

TheBloke_Mixtral-8x7B-Instruct-v0.1-GGUF - [pass]
`"1 sister - her other female sibling."`

TheBloke_Mixtral-8x7B-MoE-RP-Story-GGU - [pass]
`"Sally has 1 sister."`

TheBloke_Mixtral-8x7B-v0.1-GGUF - [pass]
`"1 sister – her other female sibling!"`

TheBloke_Mixtral-SlimOrca-8x7B-GGUF - [pass]
`"1 sister"` 

TheBloke_mixtralnt-4x7b-test-GGUF - [pass] 
`"1 sister"`

TheBloke_MythoMax-L2-33B-GPTQ - [fail]
With minimal temp, gave a few different random odd answers.

TheBloke_Nethena-20B-Glued-GPTQ - [pass]
`"1"`

TheBloke_NeuralHermes-2.5-Mistral-7B-GPTQ - [pass]
`" She has 1 sister."`

TheBloke_NeuralOrca-7B-v1-GPTQ - [pass] 
`"Sally has 1 sister."`

TheBloke_neural-chat-7B-v3-3-GPTQ - [pass]
`"The answer is: 1"`

TheBloke_OpenAssistant-Llama2-13B-Orca-8K-3319-GPTQ - [ 1/4 ? ]
It took 4 tries to get - `"Sally has 1 sister."`

TheBloke_openbuddy-mixtral-8x7b-v15.1-GGUF - [pass]
`"One sister."`

TheBloke_openchat-3.5-1210-GPTQ - [pass]
 `"Sally has 1 sister."`

TheBloke_OpenHermes-2.5-Mistral-7B-GPTQ - [pass]
`"Sally has 1 sister."`

TheBloke_OpenHermes-2.5-neural-chat-v3-3-Slerp-GPTQ - [pass]
`"Sally has 1 sister."` 

TheBloke_OpenZephyrChat-GPTQ - [pass]
gives correct answer, but does prattle on...
`"Sally has only one sister."`

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

TheBloke_Synthia-MoE-v3-Mixtral-8x7B-GGUF - [pass]
`"1 sister because she has only one actual biological sister which is her own self since they are all from the same two parents."`

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



