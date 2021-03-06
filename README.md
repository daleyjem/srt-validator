# srtValidator

This library takes in a string and validates it against the SRT spec.

## SRT

SRT is a basic subtitle format. It consists of four parts, all in text:

1. A number indicating which subtitle it is in the sequence.
2. The time that the subtitle should appear on the screen, and then disappear.
3. The subtitle itself.
4. A blank line indicating the start of a new subtitle.

Subtitles are numbered sequentially, starting at 1. The timecode format used is **hours:minutes:seconds,milliseconds** with time units fixed to two zero-padded digits and fractions fixed to three zero-padded digits (00:00:00,000).

A sample SRT looks like this:

```
1
00:02:17,440 --> 00:02:20,375
Senator, we're making
our final approach into Coruscant.

2
00:02:20,476 --> 00:02:22,501
Very good, Lieutenant.
```

## [How to debug](how-to-debug.md)

## Introductory Examples

This library exposes just one function which takes a string as an input. The output is an array of error objects. If the array is empty then there are no errors and the SRT is valid.

### Valid SRT

```
import srtValidator from 'srt-validator';

const srtString =
`1
00:02:17,440 --> 00:02:20,375
Senator, we're making
our final approach into Coruscant.

2
00:02:20,476 --> 00:02:22,501
Very good, Lieutenant.`;

srtValidator(srtString);
```

This will return:

```
[]
```

### Invalid SRT

```
import srtValidator from 'srt-validator';

const srtString =
`1
02:01:17,440 --> 02:00:20,375
Forget it, Jake.
It's Chinatown.`;

srtValidator(srtString);
```

This will return:

```
[{
  errorCode: 'validatorErrorStartTime',
  lineNumber: 2,
  message: 'start time should be less than end time',
  validator: 'CaptionTimeSpanValidator',
}]
```

## Types of Errors

### Sequence number

When the sequence number is not an integer:

```
Network
00:00:00,000 --> 00:00:00,001
I'm as mad as hell, and I'm not going to take this anymore!"
```

```
{
  errorCode: 'parserErrorInvalidSequenceNumber',
  lineNumber: 1,
  message:
    'Expected Integer for sequence number: Network',
}
```

When the first sequence is not 1:

```
2
00:00:00,000 --> 00:00:00,001
Louis, I think this is the beginning of a beautiful friendship.
```

```
{
  errorCode: 'validatorErrorSequenceNumberStart',
  lineNumber: 1,
  message: 'number of sequence need to start with 1',
  validator: 'LineNumberValidator',
}
```

When the sequences are not in order:

```
1
00:00:00,000 --> 00:00:00,001
You know how to whistle, don't you, Steve?

3
00:00:00,001 --> 00:00:00,002
You just put your lips together and blow.
```

```
{
  errorCode: 'validatorErrorSequenceNumberIncrement',
  lineNumber: 5,
  message: 'number of sequence need to increment by 1',
  validator: 'LineNumberValidator',
}
```

When the sequence number is missing:

```
1
00:00:01,000 --> 00:00:02,000
Badges? We ain't got no badges! We don't need no badges!


00:00:02,000 --> 00:00:03,000
I don't have to show you any stinking badges!
```

```
{
  errorCode: 'parserErrorMissingSequenceNumber',
  lineNumber: 5,
  message: 'Missing sequence number',
},
```

### Time Span

When the start of a sequence is after it ends:

```
1
00:00:00,000 --> 00:00:00,001
You've got to ask yourself one question: "Do I feel lucky?"

2
00:00:00,002 --> 00:00:00,001
Well, do ya, punk?
```

```
{
  errorCode: 'validatorErrorStartTime',
  lineNumber: 6,
  message: 'start time should be less than end time',
  validator: 'CaptionTimeSpanValidator',
}
```

When the start of one sequence is before the previous sequence's end:

```
1
00:00:00,000 --> 00:00:00,001
One morning I shot an elephant in my pajamas.

2
00:00:00,001 --> 00:00:00,002
How he got in my pajamas...

3
00:00:00,001 --> 00:00:00,003
I don't know.
```

```
{
  errorCode: 'validatorErrorEndTime',
  lineNumber: 10,
  message: 'start time should be less than previous end time',
  validator: 'CaptionTimeSpanValidator',
}
```

When the time span is missing:

```
1

There's no crying in baseball!
```

```
{
  errorCode: 'parserErrorMissingTimeSpan',
  lineNumber: 2,
  message: 'Missing time span',
}
```

### Timestamp

When the timestamp isn't fixed to two zero-padded digits and fractions fixed to three zero-padded digits:

```
1
0:0:0,0 --> 0:0:0,1
A boy's best friend is his mother.
```

```
{
  errorCode: 'parserErrorInvalidTimeStamp',
  lineNumber: 2,
  message: 'Invalid time stamp: 0:0:0,0',
}
```

When the fractional seperator is a period and not a comma:

```
1
00:00:00.000 --> 00:00:00.001
Mrs. Robinson, you're trying to seduce me. Aren't you?
```

```
{
  errorCode: 'parserErrorInvalidTimeStamp',
  lineNumber: 2,
  message: 'Invalid time stamp: 00:00:00.000',
}
```

### Captions text

When the caption text is missing:

```
1
00:00:00,000 --> 00:00:00,001

2
00:00:00,001 --> 00:00:00,002
My mother thanks you. My father thanks you. My sister thanks you. And I thank you.
```

```
{
  errorCode: 'parserErrorMissingText',
  lineNumber: 3,
  message: 'Missing caption text',
}
```
