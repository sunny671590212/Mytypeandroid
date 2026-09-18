package com.mytype.app;

import android.app.Activity;
import android.content.Context;
import android.content.Intent;
import android.graphics.*;
import android.graphics.drawable.GradientDrawable;
import android.os.Bundle;
import android.view.*;
import android.view.inputmethod.InputMethodManager;
import android.widget.*;
import java.io.File;
import java.io.FileOutputStream;
import java.util.*;

public class MainActivity extends Activity {
    LinearLayout root, keyboard;
    TextView target, status, progress, output;
    HandwritingView pad;
    int index;
    String letters = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    StringBuilder typed = new StringBuilder();
    android.content.SharedPreferences prefs;

    int dp(float v) {
        return (int)(v * getResources().getDisplayMetrics().density + 0.5f);
    }

    TextView tv(String s, float sp) {
        TextView t = new TextView(this);
        t.setText(s);
        t.setTextSize(sp);
        t.setTextColor(Color.WHITE);
        t.setPadding(dp(10), dp(8), dp(10), dp(8));
        return t;
    }

    GradientDrawable bg(int c) {
        GradientDrawable g = new GradientDrawable();
        g.setColor(c);
        g.setCornerRadius(dp(18));
        g.setStroke(dp(1), Color.rgb(70, 70, 70));
        return g;
    }

    @Override
    public void onCreate(Bundle b) {
        super.onCreate(b);
        prefs = getSharedPreferences("mytype", 0);
        index = prefs.getInt("count", 0);
        build();
    }

    void build() {
        ScrollView scroll = new ScrollView(this);

        root = new LinearLayout(this);
        root.setOrientation(LinearLayout.VERTICAL);
        root.setPadding(dp(16), dp(18), dp(16), dp(28));
        root.setBackgroundColor(Color.BLACK);

        scroll.addView(root);
        setContentView(scroll);

        TextView title = tv("MyType ✍️", 30);
        title.setTypeface(Typeface.DEFAULT, Typeface.BOLD);
        root.addView(title);

        root.addView(tv("Your handwriting. Your keyboard.", 15));

        LinearLayout card = new LinearLayout(this);
        card.setOrientation(LinearLayout.VERTICAL);
        card.setPadding(dp(16), dp(16), dp(16), dp(16));
        card.setBackground(bg(Color.rgb(40, 40, 40)));

        root.addView(card, lp(-1, -2, 0f, 0, 0, 0, dp(14)));

        TextView h = tv("1. Create your handwriting", 21);
        h.setTypeface(Typeface.DEFAULT, Typeface.BOLD);
        card.addView(h);

        card.addView(tv(
                "Write each character naturally. Saved letters stay on this phone.",
                14
        ));

        target = tv(
                index < 26 ? String.valueOf(letters.charAt(index)) : "✓",
                54
        );
        target.setGravity(Gravity.CENTER);
        card.addView(target, new LinearLayout.LayoutParams(-1, dp(88)));

        pad = new HandwritingView(this);
        card.addView(pad, new LinearLayout.LayoutParams(-1, dp(210)));

        LinearLayout row = new LinearLayout(this);

        Button clear = new Button(this);
        clear.setText("Clear");

        Button save = new Button(this);
        save.setText(
                index < 26
                        ? "Save " + letters.charAt(index) + " →"
                        : "A–Z Complete"
        );

        row.addView(clear, new LinearLayout.LayoutParams(0, dp(54), 1));
        row.addView(save, new LinearLayout.LayoutParams(0, dp(54), 1));
        card.addView(row);

        status = tv(
                index == 0
                        ? "A likho → Save A dabao."
                        : (index < 26
                        ? "Saved letters: " + index + ". Ab " +
                        letters.charAt(index) + " likho."
                        : "🎉 A–Z complete! Keyboard ready."),
                14
        );

        status.setGravity(Gravity.CENTER);
        card.addView(status);

        progress = tv(index + " / 26", 13);
        progress.setGravity(Gravity.CENTER);
        card.addView(progress);

        clear.setOnClickListener(v -> pad.clear());
        save.setOnClickListener(v -> saveLetter());

        LinearLayout kcard = new LinearLayout(this);
        kcard.setOrientation(LinearLayout.VERTICAL);
        kcard.setPadding(dp(16), dp(16), dp(16), dp(16));
        kcard.setBackground(bg(Color.rgb(40, 40, 40)));

        root.addView(kcard, lp(-1, -2, 0f, 0, 0, 0, dp(14)));

        TextView kh = tv("2. Live handwriting preview", 21);
        kh.setTypeface(Typeface.DEFAULT, Typeface.BOLD);
        kcard.addView(kh);

        kcard.addView(
                tv("Tap a key. Your saved glyph appears below.", 14)
        );

        keyboard = new LinearLayout(this);
        keyboard.setOrientation(LinearLayout.VERTICAL);
        kcard.addView(keyboard);

        buildPreviewKeyboard();

        LinearLayout kr = new LinearLayout(this);

        Button space = new Button(this);
        space.setText("Space");

        Button del = new Button(this);
        del.setText("⌫ Delete");

        Button clearText = new Button(this);
        clearText.setText("Clear");

        kr.addView(space, new LinearLayout.LayoutParams(0, dp(54), 1));
        kr.addView(del, new LinearLayout.LayoutParams(0, dp(54), 1));
        kr.addView(clearText, new LinearLayout.LayoutParams(0, dp(54), 1));

        kcard.addView(kr);

        space.setOnClickListener(v -> {
            typed.append(' ');
            renderOutput();
        });

        del.setOnClickListener(v -> {
            if (typed.length() > 0) {
                typed.deleteCharAt(typed.length() - 1);
                renderOutput();
            }
        });

        clearText.setOnClickListener(v -> {
            typed.setLength(0);
            renderOutput();
        });

        LinearLayout ocard = new LinearLayout(this);
        ocard.setOrientation(LinearLayout.VERTICAL);
        ocard.setPadding(dp(16), dp(16), dp(16), dp(16));
        ocard.setBackground(bg(Color.rgb(40, 40, 40)));

        root.addView(ocard, lp(-1, -2, 0f, 0, 0, 0, dp(14)));

        TextView oh = tv("3. Your actual handwriting", 21);
        oh.setTypeface(Typeface.DEFAULT, Typeface.BOLD);
        ocard.addView(oh);

        output = tv("Keyboard se letters dabao.", 16);
        output.setTextColor(Color.DKGRAY);
        output.setGravity(Gravity.CENTER_VERTICAL);
        output.setPadding(dp(14), dp(14), dp(14), dp(14));
        output.setBackground(bg(Color.WHITE));

        ocard.addView(
                output,
                new LinearLayout.LayoutParams(-1, dp(130))
        );

        Button demo = new Button(this);
        demo.setText("Try HELLO");
        ocard.addView(demo);

        demo.setOnClickListener(v -> {
            typed.setLength(0);
            typed.append("HELLO");
            renderOutput();
        });

        LinearLayout scard = new LinearLayout(this);
        scard.setOrientation(LinearLayout.VERTICAL);
        scard.setPadding(dp(16), dp(16), dp(16), dp(16));
        scard.setBackground(bg(Color.rgb(40, 40, 40)));

        root.addView(scard, lp(-1, -2, 0f, 0, 0, 0, 0));

        TextView sh = tv("4. Use MyType in Android", 21);
        sh.setTypeface(Typeface.DEFAULT, Typeface.BOLD);
        scard.addView(sh);

        Button settings = new Button(this);
        settings.setText("⚙ Enable / select MyType keyboard");
        scard.addView(settings);

        settings.setOnClickListener(v ->
                startActivity(
                        new Intent("android.settings.INPUT_METHOD_SETTINGS")
                )
        );

        Button pick = new Button(this);
        pick.setText("⌨️ Choose MyType now");
        scard.addView(pick);

        pick.setOnClickListener(v ->
                ((InputMethodManager) getSystemService(INPUT_METHOD_SERVICE))
                        .showInputMethodPicker()
        );

        scard.addView(
                tv(
                        "Note: Android text fields receive normal text. " +
                        "The handwriting preview/image is for sharing or " +
                        "apps that support image content.",
                        13
                )
        );
    }

    LinearLayout.LayoutParams lp(
            int w,
            int h,
            float a,
            int l,
            int t,
            int r,
            int b
    ) {
        LinearLayout.LayoutParams p =
                new LinearLayout.LayoutParams(w, h, a);

        p.setMargins(l, t, r, b);
        return p;
    }

    void saveLetter() {
        if (index >= 26) return;

        if (pad.isBlank()) {
            status.setText(
                    "✍️ Pehle " + letters.charAt(index) + " likho."
            );
            return;
        }

        String ch = String.valueOf(letters.charAt(index));

        try {
            File f = new File(getFilesDir(), ch + ".png");
            FileOutputStream o = new FileOutputStream(f);

            pad.bitmap.compress(
                    Bitmap.CompressFormat.PNG,
                    100,
                    o
            );

            o.close();

            index++;

            prefs.edit()
                    .putInt("count", index)
                    .apply();

            pad.clear();

            target.setText(
                    index < 26
                            ? String.valueOf(letters.charAt(index))
                            : "✓"
            );

            progress.setText(index + " / 26");

            status.setText(
                    index < 26
                            ? "Saved " + ch + " ✓ Ab " +
                            letters.charAt(index) + " likho."
                            : "🎉 A–Z complete! Keyboard ready."
            );

            buildPreviewKeyboard();

        } catch (Exception e) {
            status.setText("Save error: " + e.getMessage());
        }
    }

    void buildPreviewKeyboard() {
        keyboard.removeAllViews();

        String[] rows = {
                "QWERTYUIOP",
                "ASDFGHJKL",
                "ZXCVBNM"
        };

        for (String s : rows) {
            LinearLayout r = new LinearLayout(this);

            for (char c : s.toCharArray()) {
                Button b = new Button(this);
                b.setText(String.valueOf(c));
                b.setTextSize(15);

                File f = new File(
                        getFilesDir(),
                        c + ".png"
                );

                if (f.exists()) {
                    Bitmap bm =
                            BitmapFactory.decodeFile(
                                    f.getAbsolutePath()
                            );

                    ImageView iv = new ImageView(this);
                    iv.setImageBitmap(bm);
                    iv.setScaleType(
                            ImageView.ScaleType.CENTER_INSIDE
                    );

                    r.addView(
                            iv,
                            new LinearLayout.LayoutParams(
                                    0,
                                    dp(58),
                                    1
                            )
                    );

                    iv.setOnClickListener(v -> {
                        typed.append(c);
                        renderOutput();
                    });

                } else {
                    r.addView(
                            b,
                            new LinearLayout.LayoutParams(
                                    0,
                                    dp(58),
                                    1
                            )
                    );

                    b.setOnClickListener(v -> {
                        typed.append(c);
                        renderOutput();
                    });
                }
            }

            keyboard.addView(r);
        }
    }

    void renderOutput() {
        if (typed.length() == 0) {
            output.setText("Keyboard se letters dabao.");
            return;
        }

        output.setText(typed.toString());
        output.setTypeface(
                Typeface.create(
                        Typeface.SERIF,
                        Typeface.NORMAL
                )
        );
    }

    public static class HandwritingView extends View {

        Paint p = new Paint(3);
        Bitmap bitmap;
        Canvas c;
        float lx, ly;
        boolean down = false;

        public HandwritingView(Context x) {
            super(x);

            p.setColor(Color.BLACK);
            p.setStrokeWidth(9);
            p.setStrokeCap(Paint.Cap.ROUND);

            setBackgroundColor(Color.WHITE);
        }

        protected void onSizeChanged(
                int w,
                int h,
                int ow,
                int oh
        ) {
            bitmap = Bitmap.createBitmap(
                    Math.max(1, w),
                    Math.max(1, h),
                    Bitmap.Config.ARGB_8888
            );

            c = new Canvas(bitmap);
            c.drawColor(Color.WHITE);
        }

        protected void onDraw(Canvas canvas) {
            super.onDraw(canvas);

            if (bitmap != null) {
                canvas.drawBitmap(bitmap, 0, 0, null);
            }
        }

        public boolean onTouchEvent(
                android.view.MotionEvent e
        ) {
            float x = e.getX();
            float y = e.getY();

            switch (e.getAction()) {

                case 0:
                    down = true;
                    lx = x;
                    ly = y;
                    c.drawCircle(x, y, 5, p);
                    invalidate();
                    return true;

                case 2:
                    if (down) {
                        c.drawLine(lx, ly, x, y, p);
                        lx = x;
                        ly = y;
                        invalidate();
                    }
                    return true;

                case 1:
                    down = false;
                    return true;
            }

            return true;
        }

        void clear() {
            if (c != null) {
                c.drawColor(Color.WHITE);
                invalidate();
            }
        }

        boolean isBlank() {
            Bitmap b = bitmap;

            if (b == null) return true;

            for (
                    int i = 0;
                    i < b.getWidth();
                    i += Math.max(1, b.getWidth() / 50)
            ) {
                for (
                        int j = 0;
                        j < b.getHeight();
                        j += Math.max(1, b.getHeight() / 50)
                ) {
                    int pixel = b.getPixel(i, j);

                    int r = Color.red(pixel);

                    if (r < 240) {
                        return false;
                    }
                }
            }

            return true;
        }
    }
}
