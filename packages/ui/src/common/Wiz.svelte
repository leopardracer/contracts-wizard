<script context="module" lang="ts">
  import { SvelteComponentTyped } from 'svelte';
  import Wiz from './Wiz.svelte';

  export const mergeAiAssistanceOptions = <
    TLanguage extends AiAssistantLanguage,
    TOption extends Partial<
      Record<AiAssistantFunctionName, Record<keyof AiAssistantContractsOptions<TLanguage>, unknown>>
    >,
  >(
    previousOptions: TOption,
    aiFunctionCall: AiFunctionCall<TLanguage>,
  ): TOption =>
    Object.keys(previousOptions).reduce(
      (acc, currentKey) => {
        if (aiFunctionCall.name === currentKey)
          //@ts-expect-error currentKey can safely access acc has it was created from previousOptions with Object.key and acc initial value is also previousOptions
          return { ...acc, [currentKey]: { ...acc[currentKey], ...aiFunctionCall.arguments } };
        else return acc;
      },
      { ...previousOptions },
    );

  export function createWiz<TLanguage extends AiAssistantLanguage>() {
    return Wiz as unknown as typeof SvelteComponentTyped<
      {
        language: TLanguage;
        currentOpts?: Record<keyof AiAssistantContractsOptions<TLanguage>, unknown>;
        currentCode: string;
        experimentalContracts?: AiAssistantFunctionName<TLanguage>[];
        sampleMessages?: string[];
      },
      { 'function-call-response': CustomEvent<AiFunctionCall<TLanguage>> }
    >;
  }
</script>

<script lang="ts">
  import UserAvatar from './icons/UserAvatar.svelte';
  import WizAvatar from './icons/WizAvatar.svelte';
  import WizIcon from './icons/WizIcon.svelte';
  import XIcon from './icons/XIcon.svelte';
  import { nanoid } from 'nanoid';
  import MinimizeIcon from './icons/MinimizeIcon.svelte';
  import MaximizeIcon from './icons/MaximizeIcon.svelte';
  import HelpTooltip from './HelpTooltip.svelte';
  import type {
    AiAssistantContractsOptions,
    AiAssistantFunctionName,
    AiAssistantLanguage,
    AiChatBodyRequest,
    AiFunctionCall,
    AiFunctionCallResponse,
    Chat,
  } from '../../api/ai-assistant/types/assistant';
  import { createEventDispatcher } from 'svelte';

  const apiHost = process.env.API_HOST;

  // If adding more props make sure to also update createWiz at the top
  export let language: AiAssistantLanguage;
  export let currentOpts: AiAssistantContractsOptions | undefined;
  export let currentCode: string;
  export let experimentalContracts: AiAssistantFunctionName[] = [];
  export let sampleMessages: string[] = [];

  const dispatch = createEventDispatcher<{ 'function-call-response': AiFunctionCall }>();

  const chatId = nanoid();
  let inProgress = false;
  let currentMessage = '';
  let showing: boolean = false;
  let expanded: boolean = false;
  let messages: Chat[] = [
    {
      role: 'assistant',
      content:
        'I can also edit Wizard settings directly, so from time to time I will update those based on your input.',
    },
    {
      role: 'assistant',
      content: 'Wiz here 👋. Feel free to ask any questions you have about smart contract development.',
    },
  ];

  const addMessage = (message: Chat) => {
    messages = [
      {
        role: message.role,
        content: message.content,
      },
      ...messages,
    ];
  };

  const stringifyIfNumber = (functionCallValue: unknown) =>
    typeof functionCallValue === 'number' ? functionCallValue.toString() : functionCallValue;

  const buildOptionsFromArguments = (functionCallArguments: string) => {
    const toCallArguments: AiFunctionCall['arguments'] = JSON.parse(functionCallArguments);

    return Object.entries(toCallArguments).reduce(
      (builtOptions, [functionCallKey, functionCallValue]) => ({
        ...builtOptions,
        [functionCallKey]: stringifyIfNumber(functionCallValue),
      }),
      {} as AiAssistantContractsOptions,
    );
  };

  const submitChat = (message: Chat) => {
    inProgress = true;
    addMessage(message);
    input = '';

    const chat = messages.slice(0, messages.length - 2).reverse();

    const aiAssistantRequest: AiChatBodyRequest = {
      language,
      currentOpts,
      currentCode,
      chatId,
      messages: chat,
    };

    fetch(`${apiHost}/ai`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(aiAssistantRequest),
    })
      .then(async response => {
        const reader = response.body?.getReader();
        const decoder = new TextDecoder('utf-8');
        let result = '';
        if (reader) {
          while (true) {
            const { done, value } = await reader.read();
            if (done) break;

            // Massage and parse the chunk of data
            const chunk = decoder.decode(value);
            result += chunk;

            if (result.startsWith('{"function_call":')) {
              currentMessage = 'Building contract...';
            } else {
              currentMessage = result;
            }
          }
        }

        currentMessage = '';
        if (result.startsWith('{"function_call":')) {
          try {
            const { function_call }: AiFunctionCallResponse = JSON.parse(result);

            addMessage({
              role: 'assistant',
              content: `Updated Wizard using ${function_call.name}${
                experimentalContracts.includes(function_call.name) ? ' (Experimental).' : '.'
              }`,
            });

            dispatch('function-call-response', {
              name: function_call.name,
              arguments: buildOptionsFromArguments(function_call.arguments),
            });
          } catch (e) {
            addMessage({
              role: 'assistant',
              content: 'Error executing function. See console for details.',
            });
            console.log(e);
          }
        } else {
          addMessage({
            role: 'assistant',
            content: result,
          });
        }
        inProgress = false;
      })
      .catch(error => {
        addMessage({
          role: 'assistant',
          content: 'Error calling API. See console for details.',
        });
        console.log(error);
        inProgress = false;
      });
  };

  let input = '';
  let listener = (e: KeyboardEvent) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      if (inProgress || input.trim().length === 0) {
        return;
      }

      submitChat({
        role: 'user',
        content: input.trim(),
      });
    }
  };
</script>

<div class="absolute bottom-8 right-8 h-[calc(100%-188px)]">
  <div
    class={`${showing ? '' : 'hidden'} ${expanded ? 'w-[40rem]' : 'w-80'} max-w-[400px] min-h-[200px] absolute flex flex-col-reverse right-0 bottom-[3.75rem] border-0 shadow-xl bg-gray-50 rounded-md animate-fade-up max-h-full overflow-y-auto z-20`}
  >
    <div class={`flex flex-col-reverse gap-3 overflow-y-auto p-4 h-[calc(100%-800px)]`}>
      <div class="w-full flex items-center justify-between gap-2">
        <textarea
          bind:value={input}
          on:keypress={listener}
          placeholder="Ask me anything..."
          class="w-full text-sm shadow-lg h-32 p-4 rounded-md outline-none border-0 resize-none"
        />
        {#if input.length === 0 && messages.length < 3}
          <div class="absolute left-4 right-4 bottom-4 overflow-x-auto h-14 p-2 flex items-center gap-2">
            {#each sampleMessages as message}
              <button
                class="rounded-md border-0 p-2 h-full text-xs flex-none bg-gray-100 shadow-sm flex items-center cursor-pointer text-center hover:bg-gray-200"
                on:click={() => {
                  submitChat({
                    role: 'user',
                    content: message,
                  });
                }}
              >
                {message}
              </button>
            {/each}
          </div>
        {/if}
      </div>

      {#if currentMessage.length > 0}
        <div class="flex gap-2">
          <div class="relative h-[36px] w-[36px] flex-none">
            <div class="absolute w-[36px] h-[36px] glimmery rounded-full animate-spin-slow"></div>
            <div
              class="absolute m-[2px] flex items-center justify-center w-[32px] h-[32px] bg-gray-900 text-gray-50 rounded-full"
            >
              <WizAvatar />
            </div>
          </div>
          <div class="text-sm bg-gray-200 rounded-md p-4">{currentMessage}</div>
        </div>
      {/if}

      {#each messages as message}
        <div class="flex gap-2">
          <div class="relative h-[36px] w-[36px] flex-none">
            <div class="absolute w-[36px] h-[36px] glimmery rounded-full animate-spin-slow"></div>
            {#if message.role === 'assistant'}
              <div
                class="absolute m-[2px] flex items-center justify-center w-[32px] h-[32px] bg-gray-900 text-gray-50 rounded-full"
              >
                <WizAvatar />
              </div>
            {:else if message.role === 'user'}
              <div class="absolute m-[2px] flex items-center justify-center w-[32px] h-[32px] bg-gray-100 rounded-full">
                <UserAvatar />
              </div>
            {/if}
          </div>
          {#if message.role === 'assistant'}
            {#if message.content}
              <div class="text-sm bg-gray-200 rounded-md p-4">{message.content}</div>
            {:else}
              <div class="text-sm bg-gray-200 rounded-md font-bold p-4 text-gray-500">{'Called a function'}</div>
            {/if}
          {:else if message.role === 'user'}
            <div class="text-sm bg-gray-100 rounded-md p-4">{message.content}</div>
          {/if}
        </div>
      {/each}
    </div>
    <div
      class="flex items-center justify-between border-0 border-b border-solid border-gray-200 p-4 font-bold text-gray-600"
    >
      AI Assistant
      <div class="flex items-center gap-2">
        <div class="tooltip-container">
          <HelpTooltip placement="bottom">
            The AI Assistant Wiz is powered by the <a
              target="_blank"
              rel="noopener noreferrer"
              href="https://openai.com/blog/openai-api">OpenAI API</a
            >, which utilizes AI/ML and therefore may produce inaccurate information.
            <br /><br />
            You should always review any information produced by Wiz to ensure that any results are accurate and suit your
            purposes.
          </HelpTooltip>
        </div>
        <button
          class="shadow-xl rotate-45 border-none bg-gray-400 rounded-md h-6 w-6 text-white cursor-pointer"
          on:click={() => {
            expanded = !expanded;
          }}
        >
          {#if expanded}
            <MinimizeIcon />
          {:else}
            <MaximizeIcon />
          {/if}
        </button>
      </div>
    </div>
  </div>
  {#if showing}
    <button
      class="text-sm z-20 absolute flex right-0 bottom-1 items-center justify-center border-0 shadow-xl bg-gray-50 h-10 w-10 p-4 rounded-full cursor-pointer hover:bg-white"
      on:click={() => {
        showing = false;
      }}
    >
      <div class="animate-fade-in">
        <XIcon />
      </div>
    </button>
  {:else}
    <button
      class="text-sm z-20 absolute flex right-0 bottom-1 items-center justify-center border-0 shadow-xl bg-gray-50 h-10 w-32 p-4 rounded-full cursor-pointer hover:bg-white"
      on:click={() => {
        showing = true;
      }}
    >
      <div class="animate-fade-in">
        <WizIcon /> AI Assistant
      </div>
    </button>
  {/if}
</div>
