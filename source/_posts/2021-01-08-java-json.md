---
title: Java实现JSON解析
date: 2021-01-08 20:45:41
categories: 技术分享
tags:
- Java
---


# 测试
```java
public class Main {

    public static void main(String[] args) throws Exception {
        String text = 
                "{\n" +
                "  \"transformRequest\":\n" +
                "  {},\n" +
                "  \"transformResponse\":\n" +
                "  {},\n" +
                "  \"timeout\": 30000,\n" +
                "  \"xsrfHeaderName\": [\"X-XSRF-TOKEN\",\"X-XSRF-TOKEN2\",\"X-XSRF-TOKEN3\"],\n" +
                "  \"maxContentLength\": -1,\n" +
                "  \"headers\":\n" +
                "  {\n" +
                "      \"Accept\": \"application/json, text/plain, */*\",\n" +
                "      \"Content-Type\": \"application/json;charset=UTF-8\"\n" +
                "  },\n" +
                "  \"data\": \"{\\\"page\\\":1,\\\"limit\\\":10}\"\n" +
                "}";

        System.out.println(JSONUtils.formatText(text));
        System.out.println("===================");
        System.out.println(JSON.toJSONString(JSONUtils.parseText(text), true));
    }
}
```

# 实现
```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class JSONUtils {

    public static Map<String, Object> parseObjectText(String text) throws Exception {
        Object o = parseText(text);
        if (!(o instanceof Map)) {
            throw new Exception();
        }
        return (Map<String, Object>) o;
    }

    public static List<Object> parseArrayText(String text) throws Exception {
        Object o = parseText(text);
        if (!(o instanceof List)) {
            throw new Exception();
        }
        return (List<Object>) o;
    }

    public static Object parseText(String text) throws Exception {
        return parseTokens(parseTextToTokens(text), 0).data;
    }

    public static String formatText(String text) throws Exception {
        return formatTokens(parseTextToTokens(text));
    }

    private static Token getFirstToken(String text, TokenType type) {
        Matcher matcher = type.pattern.matcher(text);
        if (!matcher.find()) {
            return null;
        }
        return new Token(type, matcher.group(0));
    }

    private static String transString(String text) {
        StringBuilder val = new StringBuilder();
        for (int i = 1; i < text.length() - 1; i++) {
            if (text.charAt(i) != '\\') {
                val.append(text.charAt(i));
            } else {
                switch (text.charAt(i + 1)) {
                    case 'n':
                        val.append('\n');
                        break;
                    case 't':
                        val.append('\t');
                        break;
                    default:
                        val.append(text.charAt(i + 1));
                        break;
                }
                i++;
            }
        }
        return val.toString();
    }

    private static TokensParseResult parseTokens(List<Token> tokens, int index) throws Exception {
        switch (tokens.get(index).type) {
            case OBJECT_OPEN: {
                index++;
                Map<String, Object> data = new HashMap<>();
                while (tokens.get(index).type != TokenType.OBJECT_CLOSE) {
                    if (tokens.get(index).type == TokenType.STRING && tokens.get(index + 1).type == TokenType.COLON) {
                        TokensParseResult item = parseTokens(tokens, index + 2);
                        data.put(transString(tokens.get(index).value), item.data);
                        index = item.nextIndex;
                        if (tokens.get(index).type == TokenType.SPLIT) {
                            index++;
                        }
                    } else {
                        throw new Exception();
                    }
                }
                return new TokensParseResult(data, index + 1);
            }
            case ARRAY_OPEN: {
                index++;
                List<Object> data = new ArrayList<>();
                while (tokens.get(index).type != TokenType.ARRAY_CLOSE) {
                    TokensParseResult item = parseTokens(tokens, index);
                    data.add(item.data);
                    index = item.nextIndex;
                    if (tokens.get(index).type == TokenType.SPLIT) {
                        index++;
                    }
                }
                return new TokensParseResult(data, index + 1);

            }
            case STRING:
                return new TokensParseResult(transString(tokens.get(index).value), index + 1);
            case NUMBER:
                return new TokensParseResult(Double.parseDouble(tokens.get(index).value), index + 1);
            case BOOLEAN:
                return new TokensParseResult(Boolean.parseBoolean(tokens.get(index).value), index + 1);
            case NULL:
                return new TokensParseResult(null, index + 1);
            default:
                throw new Exception();
        }
    }

    private static String enter(int depth) {
        StringBuilder val = new StringBuilder("\n");
        for (int i = 0; i < depth; i++) {
            val.append("    ");
        }
        return val.toString();
    }

    private static String formatTokens(List<Token> tokens) {
        StringBuilder format = new StringBuilder();
        int depth = 0;
        for (Token token : tokens) {
            if (token.type == TokenType.OBJECT_CLOSE || token.type == TokenType.ARRAY_CLOSE) {
                format.append(enter(--depth));
            }
            format.append(token.value);
            if (token.type == TokenType.OBJECT_OPEN || token.type == TokenType.ARRAY_OPEN) {
                format.append(enter(++depth));
            }
            if (token.type == TokenType.SPLIT) {
                format.append(enter(depth));
            }
        }
        return format.toString();
    }

    private static List<Token> parseTextToTokens(String text) throws Exception {
        List<Token> tokens = new ArrayList<>();
        while (!text.isEmpty()) {
            Token token = null;
            for (TokenType type : TokenType.values()) {
                token = getFirstToken(text, type);
                if (token != null) {
                    break;
                }
            }
            if (token == null) {
                throw new Exception();
            }
            text = text.substring(token.value.length());
            if (token.type == TokenType.SPACE) {
                continue;
            }
            tokens.add(token);
        }
        return tokens;
    }

    private static class Token {
        private TokenType type;
        private String value;

        private Token(TokenType type, String value) {
            this.type = type;
            this.value = value;
        }
    }

    private static class TokensParseResult {
        private Object data;
        private int nextIndex;

        TokensParseResult(Object data, int nextIndex) {
            this.data = data;
            this.nextIndex = nextIndex;
        }
    }

    private enum TokenType {
        STRING("^(\"([^\"\\\\]|\\\\.)*\")"),
        NUMBER("^(-?\\d+(\\.\\d+)?)"),
        BOOLEAN("^(true|false)"),
        NULL("^(null)"),
        OBJECT_OPEN("^(\\{)"),
        OBJECT_CLOSE("^(\\})"),
        ARRAY_OPEN("^(\\[)"),
        ARRAY_CLOSE("^(\\])"),
        COLON("^(\\:)"),
        SPLIT("^(\\,)"),
        SPACE("^([\\s\\r\\n]+)");
        
        private Pattern pattern;

        TokenType(String reg) {
            this.pattern = Pattern.compile(reg);
        }
    }
}
```