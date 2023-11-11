This is a write-up of a few of the flags from the intentionally vulnerable Android app [Injured Android](https://github.com/B3nac/InjuredAndroid).

I found that this was a great way to get hands-on experience with testing an Android app with fewer unknowns compared to some other CTF apps.

# Initial Setup

Getting started I de-compiled the APK and went ahead and installed the app on our emulator.

De-compile the APK: `./jadx /InjuredAndroid/InjuredAndroid-1.0.12-release.apk -d /Decomp-InjuredAndroid --show-bad-code`

Install the app: `adb install /InjuredAndroid/InjuredAndroid-1.0.12-release.apk`

# Flag Zero: XSS Test

There is no actual flag for this one just for testing XSS. `XSSTextActivity` supplies our user input to `DisplayPostXSS`.

```jsx
package b3nac.injuredandroid;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;

/* loaded from: classes.dex */
public class XSSTextActivity extends androidx.appcompat.app.c {
    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.appcompat.app.c, androidx.fragment.app.d, androidx.activity.ComponentActivity, androidx.core.app.e, android.app.Activity
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_xsstext);
    }

    public void submitText(View view) {
        Intent intent = new Intent(this, DisplayPostXSS.class);
        intent.putExtra("com.b3nac.injuredandroid.DisplayPostXSS", ((EditText) findViewById(R.id.editText)).getText().toString());
        startActivity(intent);
    }
}
```

`DisplayPostXSS` then puts our string into a WebView instance that has JavaScript enabled without any prior sanitation. 

```jsx
package b3nac.injuredandroid;

import android.os.Bundle;
import android.webkit.WebChromeClient;
import android.webkit.WebSettings;
import android.webkit.WebView;

/* loaded from: classes.dex */
public final class DisplayPostXSS extends androidx.appcompat.app.c {
    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.appcompat.app.c, androidx.fragment.app.d, androidx.activity.ComponentActivity, androidx.core.app.e, android.app.Activity
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        WebView webView = new WebView(this);
        setContentView(webView);
        String stringExtra = getIntent().getStringExtra("com.b3nac.injuredandroid.DisplayPostXSS");
        WebSettings settings = webView.getSettings();
        d.s.d.g.d(settings, "vulnWebView.settings");
        settings.setJavaScriptEnabled(true);
        webView.setWebChromeClient(new WebChromeClient());
        webView.loadData(stringExtra, "text/html", "UTF-8");
    }
}
```

`<h1>XSS <script>alert(1)</script>!</h1>`

![xss2.png](assets/images/xss2.png)

# Flag One: Login

For the first real flag, we can view the conveniently named class `FlagOneLoginActivity` from the de-compiled APK.

```python
public final class FlagOneLoginActivity extends androidx.appcompat.app.c {
    private int w;

    /* loaded from: classes.dex */
    static final class a implements View.OnClickListener {
        a() {
        }

        @Override // android.view.View.OnClickListener
        public final void onClick(View view) {
            if (FlagOneLoginActivity.this.F() == 0) {
                Snackbar X = Snackbar.X(view, "The flag is right under your nose.", 0);
                X.Y("Action", null);
                X.N();
                FlagOneLoginActivity flagOneLoginActivity = FlagOneLoginActivity.this;
                flagOneLoginActivity.G(flagOneLoginActivity.F() + 1);
            } else if (FlagOneLoginActivity.this.F() == 1) {
                Snackbar X2 = Snackbar.X(view, "The flag is also under the GUI.", 0);
                X2.Y("Action", null);
                X2.N();
                FlagOneLoginActivity.this.G(0);
            }
        }
    }

    public final int F() {
        return this.w;
    }

    public final void G(int i) {
        this.w = i;
    }

    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.appcompat.app.c, androidx.fragment.app.d, androidx.activity.ComponentActivity, androidx.core.app.e, android.app.Activity
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_flag_one_login);
        j.j.a(this);
        C((Toolbar) findViewById(R.id.toolbar));
        ((FloatingActionButton) findViewById(R.id.fab)).setOnClickListener(new a());
    }

    public final void submitFlag(View view) {
        EditText editText = (EditText) findViewById(R.id.editText2);
        d.s.d.g.d(editText, "editText2");
        if (d.s.d.g.a(editText.getText().toString(), "F1ag_0n3")) {
            Intent intent = new Intent(this, FlagOneSuccess.class);
            new FlagsOverview().J(true);
            new j().b(this, "flagOneButtonColor", true);
            startActivity(intent);
        }
    }
}
```

Reviewing the code we find `submitFlag()` which compares our user-supplied string to a hard-coded string `F1ag_0n3`. If our string matches the hard-coded string then a new intent `FlagOneSuccess` will be triggered

```jsx
    public final void submitFlag(View view) {
        EditText editText = (EditText) findViewById(R.id.editText2);
        d.s.d.g.d(editText, "editText2");
        if (d.s.d.g.a(editText.getText().toString(), "F1ag_0n3")) {
            Intent intent = new Intent(this, FlagOneSuccess.class);
            new FlagsOverview().J(true);
            new j().b(this, "flagOneButtonColor", true);
            startActivity(intent);
        }
    }
```

# Flag Two: Exported Activities

Looking into `FlagTwoActivity` we see the hints that we need to use an exported activity. 

```jsx
package b3nac.injuredandroid;

import android.os.Bundle;
import android.view.View;
import androidx.appcompat.widget.Toolbar;
import com.google.android.material.floatingactionbutton.FloatingActionButton;
import com.google.android.material.snackbar.Snackbar;

/* loaded from: classes.dex */
public class FlagTwoActivity extends androidx.appcompat.app.c {
    int w = 0;

    public /* synthetic */ void F(View view) {
        int i = this.w;
        if (i == 0) {
            Snackbar X = Snackbar.X(view, "Key words Activity and exported.", 0);
            X.Y("Action", null);
            X.N();
            this.w++;
        } else if (i == 1) {
            Snackbar X2 = Snackbar.X(view, "Exported Activities can be accessed with adb or Drozer.", 0);
            X2.Y("Action", null);
            X2.N();
            this.w = 0;
        }
    }

    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.appcompat.app.c, androidx.fragment.app.d, androidx.activity.ComponentActivity, androidx.core.app.e, android.app.Activity
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_flag_two);
        C((Toolbar) findViewById(R.id.toolbar));
        ((FloatingActionButton) findViewById(R.id.fab)).setOnClickListener(new View.OnClickListener() { // from class: b3nac.injuredandroid.d
            @Override // android.view.View.OnClickListener
            public final void onClick(View view) {
                FlagTwoActivity.this.F(view);
            }
        });
    }
}
```

Reviewing the manifest we can find a few exported activities, however, the one we want is `b25lActivity`. 

```jsx
<activity android:name=".b25lActivity" android:exported="true" />
```

Once we open `b25lActivity` we will see that using this activity will trigger the flag for this flag activity. 

```jsx
package b3nac.injuredandroid;

import android.os.Bundle;

/* loaded from: classes.dex */
public final class b25lActivity extends androidx.appcompat.app.c {
    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.appcompat.app.c, androidx.fragment.app.d, androidx.activity.ComponentActivity, androidx.core.app.e, android.app.Activity
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_b25l);
        j.j.a(this);
        new FlagsOverview().M(true);
        new j().b(this, "flagTwoButtonColor", true);
    }
}
```

To call the activity: `adb shell am start -n b3nac.injuredandroid/b3nac.injuredandroid.b25lActivity`

![flag2.png](assets/images/flag2.png)

# Flag Three: Resources

Now onto `FlagThreeActivity` we can see that within `submitFlag()` our user-supplied string is being compared to a string that is being retrieved using `getString()`.

```python
package b3nac.injuredandroid;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import androidx.appcompat.widget.Toolbar;
import com.google.android.material.floatingactionbutton.FloatingActionButton;
import com.google.android.material.snackbar.Snackbar;

/* loaded from: classes.dex */
public final class FlagThreeActivity extends androidx.appcompat.app.c {
    private int w;

    /* loaded from: classes.dex */
    static final class a implements View.OnClickListener {
        a() {
        }

        @Override // android.view.View.OnClickListener
        public final void onClick(View view) {
            if (FlagThreeActivity.this.F() == 0) {
                Snackbar X = Snackbar.X(view, "R stands for resources.", 0);
                X.Y("Action", null);
                X.N();
                FlagThreeActivity flagThreeActivity = FlagThreeActivity.this;
                flagThreeActivity.G(flagThreeActivity.F() + 1);
            } else if (FlagThreeActivity.this.F() == 1) {
                Snackbar X2 = Snackbar.X(view, "Check .xml files.", 0);
                X2.Y("Action", null);
                X2.N();
                FlagThreeActivity.this.G(0);
            }
        }
    }

    public final int F() {
        return this.w;
    }

    public final void G(int i) {
        this.w = i;
    }

    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.appcompat.app.c, androidx.fragment.app.d, androidx.activity.ComponentActivity, androidx.core.app.e, android.app.Activity
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_flag_three);
        j.j.a(this);
        C((Toolbar) findViewById(R.id.toolbar));
        ((FloatingActionButton) findViewById(R.id.fab)).setOnClickListener(new a());
    }

    public final void submitFlag(View view) {
        EditText editText = (EditText) findViewById(R.id.editText2);
        d.s.d.g.d(editText, "editText2");
        if (d.s.d.g.a(editText.getText().toString(), getString(R.string.cmVzb3VyY2VzX3lv))) {
            Intent intent = new Intent(this, FlagOneSuccess.class);
            new FlagsOverview().L(true);
            new j().b(this, "flagThreeButtonColor", true);
            startActivity(intent);
        }
    }
}
```

Following the Androids documentation `getString()` is retrieving the string with the name of `cmVzb3VyY2VzX3lv` from `res/values/*filename*.xml`. Opening res/values we can locate `strings.xml`. A quick search for the string name and we find our flag.

```python
<string name="cmVzb3VyY2VzX3lv">F1ag_thr33</string>
```

# Flag Four: Login 2

`FlagFourActivity`

```python
package b3nac.injuredandroid;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import com.google.android.material.floatingactionbutton.FloatingActionButton;
import com.google.android.material.snackbar.Snackbar;

/* loaded from: classes.dex */
public final class FlagFourActivity extends androidx.appcompat.app.c {
    private int w;

    /* loaded from: classes.dex */
    static final class a implements View.OnClickListener {
        a() {
        }

        @Override // android.view.View.OnClickListener
        public final void onClick(View view) {
            if (FlagFourActivity.this.F() == 0) {
                Snackbar X = Snackbar.X(view, "Where is bob.", 0);
                X.Y("Action", null);
                X.N();
                FlagFourActivity flagFourActivity = FlagFourActivity.this;
                flagFourActivity.G(flagFourActivity.F() + 1);
            } else if (FlagFourActivity.this.F() == 1) {
                Snackbar X2 = Snackbar.X(view, "Classes and imports.", 0);
                X2.Y("Action", null);
                X2.N();
                FlagFourActivity.this.G(0);
            }
        }
    }

    public final int F() {
        return this.w;
    }

    public final void G(int i) {
        this.w = i;
    }

    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.appcompat.app.c, androidx.fragment.app.d, androidx.activity.ComponentActivity, androidx.core.app.e, android.app.Activity
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_flag_four);
        j.j.a(this);
        ((FloatingActionButton) findViewById(R.id.fab)).setOnClickListener(new a());
    }

    public final void submitFlag(View view) {
        EditText editText = (EditText) findViewById(R.id.editText2);
        d.s.d.g.d(editText, "editText2");
        String obj = editText.getText().toString();
        byte[] a2 = new g().a();
        d.s.d.g.d(a2, "decoder.getData()");
        if (d.s.d.g.a(obj, new String(a2, d.w.c.f2418a))) {
            Intent intent = new Intent(this, FlagOneSuccess.class);
            new FlagsOverview().I(true);
            new j().b(this, "flagFourButtonColor", true);
            startActivity(intent);
        }
    }
}
```

Once again in `submitFlag()` we can see that our user-supplied string is being compared to another string. This time it is being compared to `String(a2, d.w.c.f2418a)`. We can also see that `a2` is equal to `g().a()`. Let’s start with `d.w.c.f2418a`.

Opening up `d.w.c` we can find `f2418a` but really nothing else useful to us.

```python
package d.w;

import java.nio.charset.Charset;

/* loaded from: classes.dex */
public final class c {

    /* renamed from: a  reason: collision with root package name */
    public static final Charset f2418a;

    static {
        Charset forName = Charset.forName("UTF-8");
        d.s.d.g.d(forName, "Charset.forName(\"UTF-8\")");
        f2418a = forName;
        d.s.d.g.d(Charset.forName("UTF-16"), "Charset.forName(\"UTF-16\")");
        d.s.d.g.d(Charset.forName("UTF-16BE"), "Charset.forName(\"UTF-16BE\")");
        d.s.d.g.d(Charset.forName("UTF-16LE"), "Charset.forName(\"UTF-16LE\")");
        d.s.d.g.d(Charset.forName("US-ASCII"), "Charset.forName(\"US-ASCII\")");
        d.s.d.g.d(Charset.forName("ISO-8859-1"), "Charset.forName(\"ISO-8859-1\")");
    }
}
```

Since that was a bust let's look into `a2`. From `g()` in the `b3nac.injuredandroid` folder we find `f1468a`. Which turns out to simply be base64 decoding `NF9vdmVyZG9uZV9vbWVsZXRz`.

```python
package b3nac.injuredandroid;

import android.util.Base64;

/* loaded from: classes.dex */
public class g {

    /* renamed from: a  reason: collision with root package name */
    private byte[] f1468a = Base64.decode("NF9vdmVyZG9uZV9vbWVsZXRz", 0);

    public byte[] a() {
        return this.f1468a;
    }
}
```

Decoding the string will give us our key to trigger the flag.

```python
echo 'NF9vdmVyZG9uZV9vbWVsZXRz' | base64 --decode
4_overdone_omelets
```

# Flag Five: Exported Broadcast Receiver

From the de-compiled code for `FlagFiveActivity` we can see that an intent `com.b3nac.injuredandroid.intent.action.CUSTOM_INTENT` is being created along with a listener. Additionally the receiver is created using `ComponentName(this, FlagFiveReceiver.class)`.

```python
package b3nac.injuredandroid;

import android.content.ComponentName;
import android.content.Intent;
import android.content.IntentFilter;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import androidx.appcompat.widget.Toolbar;
import com.google.android.material.floatingactionbutton.FloatingActionButton;
import com.google.android.material.snackbar.Snackbar;

/* loaded from: classes.dex */
public class FlagFiveActivity extends androidx.appcompat.app.c {
    int w = 0;
    private FlagFiveReceiver x = new FlagFiveReceiver();

    public void F() {
        sendBroadcast(new Intent("com.b3nac.injuredandroid.intent.action.CUSTOM_INTENT"));
    }

    public /* synthetic */ void G(View view) {
        int i = this.w;
        if (i == 0) {
            Snackbar X = Snackbar.X(view, "Where is bob.", 0);
            X.Y("Action", null);
            X.N();
            this.w++;
        } else if (i == 1) {
            Snackbar X2 = Snackbar.X(view, "Classes and imports.", 0);
            X2.Y("Action", null);
            X2.N();
            this.w = 0;
        }
    }

    public /* synthetic */ void H(View view) {
        F();
    }

    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.appcompat.app.c, androidx.fragment.app.d, androidx.activity.ComponentActivity, androidx.core.app.e, android.app.Activity
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_flag_five);
        C((Toolbar) findViewById(R.id.toolbar));
        ((FloatingActionButton) findViewById(R.id.fab)).setOnClickListener(new View.OnClickListener() { // from class: b3nac.injuredandroid.c
            @Override // android.view.View.OnClickListener
            public final void onClick(View view) {
                FlagFiveActivity.this.G(view);
            }
        });
        Button button = (Button) findViewById(R.id.button9);
        new ComponentName(this, FlagFiveReceiver.class);
        getPackageManager();
        a.m.a.a.b(this).c(this.x, new IntentFilter("com.b3nac.injuredandroid.intent.action.CUSTOM_INTENT"));
        button.setOnClickListener(new View.OnClickListener() { // from class: b3nac.injuredandroid.b
            @Override // android.view.View.OnClickListener
            public final void onClick(View view) {
                FlagFiveActivity.this.H(view);
            }
        });
    }

    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.appcompat.app.c, androidx.fragment.app.d, android.app.Activity
    public void onDestroy() {
        a.m.a.a.b(this).e(this.x);
        super.onDestroy();
    }
}
```

In `FlagFiveReceiver` we can see that if `i2` is not 0 we will go into the else portion. Once there we can see that if `i2` is equal to 2 then we will trigger the flag. `i2` is equal to `f1454a` which is equal to `i` which presumably starts at zero but is increased by 1 each time the Broadcast Receiver receives the intent.

```jsx
package b3nac.injuredandroid;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.util.Log;
import android.widget.Toast;

/* loaded from: classes.dex */
public final class FlagFiveReceiver extends BroadcastReceiver {

    /* renamed from: a  reason: collision with root package name */
    private static int f1454a;

    @Override // android.content.BroadcastReceiver
    public void onReceive(Context context, Intent intent) {
        String str;
        int i;
        String e;
        String e2;
        d.s.d.g.e(context, "context");
        d.s.d.g.e(intent, "intent");
        j.j.a(context);
        int i2 = f1454a;
        if (i2 == 0) {
            StringBuilder sb = new StringBuilder();
            e = d.w.h.e("\n    Action: " + intent.getAction() + "\n\n    ");
            sb.append(e);
            e2 = d.w.h.e("\n    URI: " + intent.toUri(1) + "\n\n    ");
            sb.append(e2);
            str = sb.toString();
            d.s.d.g.d(str, "sb.toString()");
            Log.d("DUDE!:", str);
        } else {
            str = "Keep trying!";
            if (i2 != 1) {
                if (i2 != 2) {
                    Toast.makeText(context, "Keep trying!", 1).show();
                    return;
                }
                String str2 = "You are a winner " + k.a("Zkdlt0WwtLQ=");
                new FlagsOverview().H(true);
                new j().b(context, "flagFiveButtonColor", true);
                Toast.makeText(context, str2, 1).show();
                i = 0;
                f1454a = i;
            }
        }
        Toast.makeText(context, str, 1).show();
        i = f1454a + 1;
        f1454a = i;
    }
}
```

Sending a broadcast twice we can trigger the flag.

 `adb shell am broadcast -a com.b3nac.injuredandroid.intent.action.CUSTOM_INTENT`

![flag5.png](assets/images/flag5.png)

# Flag Six: Login3

`FlagSixLoginActivity` is another login-style activity. Again in `submitFlag()` our user supplied data is being compared to another object, this time it is being compared to `k.a("k3FElEG9lnoWbOateGhj5pX6QsXRNJKh///8Jxi8KXW7iDpk2xRxhQ==")`.

```jsx
package b3nac.injuredandroid;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import androidx.appcompat.widget.Toolbar;
import com.google.android.material.floatingactionbutton.FloatingActionButton;
import com.google.android.material.snackbar.Snackbar;

/* loaded from: classes.dex */
public final class FlagSixLoginActivity extends androidx.appcompat.app.c {
    private int w;

    /* loaded from: classes.dex */
    static final class a implements View.OnClickListener {
        a() {
        }

        @Override // android.view.View.OnClickListener
        public final void onClick(View view) {
            if (FlagSixLoginActivity.this.F() == 0) {
                d.s.d.g.c(view);
                Snackbar X = Snackbar.X(view, "Keys.", 0);
                X.Y("Action", null);
                X.N();
                FlagSixLoginActivity flagSixLoginActivity = FlagSixLoginActivity.this;
                flagSixLoginActivity.G(flagSixLoginActivity.F() + 1);
            } else if (FlagSixLoginActivity.this.F() == 1) {
                d.s.d.g.c(view);
                Snackbar X2 = Snackbar.X(view, "Classes.", 0);
                X2.Y("Action", null);
                X2.N();
                FlagSixLoginActivity.this.G(0);
            }
        }
    }

    public final int F() {
        return this.w;
    }

    public final void G(int i) {
        this.w = i;
    }

    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.appcompat.app.c, androidx.fragment.app.d, androidx.activity.ComponentActivity, androidx.core.app.e, android.app.Activity
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_flag_six_login);
        j.j.a(this);
        C((Toolbar) findViewById(R.id.toolbar));
        ((FloatingActionButton) findViewById(R.id.fab)).setOnClickListener(new a());
    }

    public final void submitFlag(View view) {
        EditText editText = (EditText) findViewById(R.id.editText3);
        d.s.d.g.d(editText, "editText3");
        if (d.s.d.g.a(editText.getText().toString(), k.a("k3FElEG9lnoWbOateGhj5pX6QsXRNJKh///8Jxi8KXW7iDpk2xRxhQ=="))) {
            Intent intent = new Intent(this, FlagOneSuccess.class);
            FlagsOverview.G = true;
            new j().b(this, "flagSixButtonColor", true);
            startActivity(intent);
        }
    }
}
```

Viewing `k()` in the in the `b3nac.injuredandroid` folder.

```python
package b3nac.injuredandroid;

import android.text.TextUtils;
import android.util.Base64;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.spec.InvalidKeySpecException;
import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESKeySpec;

/* loaded from: classes.dex */
public class k {

    /* renamed from: a  reason: collision with root package name */
    private static final byte[] f1472a = h.b();

    /* renamed from: b  reason: collision with root package name */
    private static final byte[] f1473b = h.a();

    public static String a(String str) {
        if (c(str)) {
            try {
                SecretKey generateSecret = SecretKeyFactory.getInstance("DES").generateSecret(new DESKeySpec(f1472a));
                byte[] decode = Base64.decode(str, 0);
                Cipher cipher = Cipher.getInstance("DES");
                cipher.init(2, generateSecret);
                return new String(cipher.doFinal(decode));
            } catch (InvalidKeyException | NoSuchAlgorithmException | InvalidKeySpecException | BadPaddingException | IllegalBlockSizeException | NoSuchPaddingException e) {
                e.printStackTrace();
            }
        } else {
            System.out.println("Not a string!");
        }
        return str;
    }

    public static String b(String str) {
        if (c(str)) {
            try {
                SecretKey generateSecret = SecretKeyFactory.getInstance("DES").generateSecret(new DESKeySpec(f1473b));
                byte[] decode = Base64.decode(str, 0);
                Cipher cipher = Cipher.getInstance("DES");
                cipher.init(2, generateSecret);
                return new String(cipher.doFinal(decode));
            } catch (InvalidKeyException | NoSuchAlgorithmException | InvalidKeySpecException | BadPaddingException | IllegalBlockSizeException | NoSuchPaddingException e) {
                e.printStackTrace();
            }
        } else {
            System.out.println("Not a string!");
        }
        return str;
    }

    public static boolean c(String str) {
        if (TextUtils.isEmpty(str)) {
            return false;
        }
        try {
            Base64.decode(str, 0);
            return true;
        } catch (Exception unused) {
            return false;
        }
    }
}
```

`"k3FElEG9lnoWbOateGhj5pX6QsXRNJKh///8Jxi8KXW7iDpk2xRxhQ=="` is being passed into `a()` as `str`. `str` is base64 decoded then passed to `String(cipher.doFinal(decode))` as `decode` where it is decrypted using a DES cipher generated using `DESKeySpec(f1472a)` from `SecretKeyFactory`.

From Oracle’s `DESKeySpec` documentation:  “`DESKeySpec(byte[] key)` Creates a DESKeySpec object using the first 8 bytes in `key` as the key material for the DES key.”

With that being said `f1472a` is going to be the `key` object for `DESKeySpec()`.

We see that the key `f1472a` is equal to `h.b()` so lets track that down.

```python
package b3nac.injuredandroid;

import android.util.Base64;

/* loaded from: classes.dex */
public class h {

    /* renamed from: a  reason: collision with root package name */
    private static byte[] f1469a = Base64.decode("Q2FwdHVyM1RoMXM=", 0);

    /* renamed from: b  reason: collision with root package name */
    private static byte[] f1470b = Base64.decode("e0NhcHR1cjNUaDFzVG9vfQ==", 0);

    /* renamed from: c  reason: collision with root package name */
    private static String f1471c = "9EEADi^^:?;FC652?5C@:5]7:C632D6:@]4@>^DB=:E6];D@?";

    /* JADX INFO: Access modifiers changed from: package-private */
    public static byte[] a() {
        return f1470b;
    }

    /* JADX INFO: Access modifiers changed from: package-private */
    public static byte[] b() {
        return f1469a;
    }

    /* JADX INFO: Access modifiers changed from: package-private */
    public static String c() {
        return f1471c;
    }
}
```

`b()` is returning `f1469a` which is simply base64 decoding `"Q2FwdHVyM1RoMXM="`.

```python
echo "Q2FwdHVyM1RoMXM=" | base64 --decode
Captur3Th1s
```

Now that we have the `DESKeySpec()` key we can reverse the encryption process. 

```python
import base64

from Crypto.Cipher import DES

key = "Captur3Th1s"
encodedString = "k3FElEG9lnoWbOateGhj5pX6QsXRNJKh///8Jxi8KXW7iDpk2xRxhQ=="

keyByte = key[:8].encode('utf-8')
stringByte = base64.b64decode(encodedString)
desReverse = DES.new(keyByte, DES.MODE_ECB)

print(desReverse.decrypt(stringByte))
```

We will then receive `b"{This_Isn't_Where_I_Parked_My_Car}\x06\x06\x06\x06\x06\x06"`.

Ignoring the trailing bytes we have our answer.

# Flag Seven: SQLite

`FlagSevenSqliteActivity`

Once we open the activity a SQLite database named `Thisisatest` is created. `contentValues.put(”title”, )` & `contentValues.put(”subtitle”, )` are being used to provide the data to be inserted into the database. 

```python
@Override // androidx.appcompat.app.c, androidx.fragment.app.d, androidx.activity.ComponentActivity, androidx.core.app.e, android.app.Activity
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_flag_seven_sqlite);
        C((Toolbar) findViewById(R.id.toolbar));
        j.j.a(this);
        H();
        ((FloatingActionButton) findViewById(R.id.fab)).setOnClickListener(new a());
        SQLiteDatabase writableDatabase = this.x.getWritableDatabase();
        ContentValues contentValues = new ContentValues();
        contentValues.put("title", Base64.decode("VGhlIGZsYWcgaGFzaCE=", 0));
        contentValues.put("subtitle", Base64.decode("MmFiOTYzOTBjN2RiZTM0MzlkZTc0ZDBjOWIwYjE3Njc=", 0));
        writableDatabase.insert("Thisisatest", null, contentValues);
        contentValues.put("title", Base64.decode("VGhlIGZsYWcgaXMgYWxzbyBhIHBhc3N3b3JkIQ==", 0));
        contentValues.put("subtitle", h.c());
        writableDatabase.insert("Thisisatest", null, contentValues);
    }
```

For the first row in the database both the `“title”` and `“subtitle”` are base64 decoded version of their associated hard-coded string.

```jsx
contentValues.put("title", Base64.decode("VGhlIGZsYWcgaGFzaCE=", 0));
contentValues.put("subtitle", Base64.decode("MmFiOTYzOTBjN2RiZTM0MzlkZTc0ZDBjOWIwYjE3Njc=", 0));
writableDatabase.insert("Thisisatest", null, contentValues);
```

Base64 decoding both we receive the following:

```jsx
title: echo "VGhlIGZsYWcgaGFzaCE=" | base64 --decode
The flag hash!

subtitle: echo "MmFiOTYzOTBjN2RiZTM0MzlkZTc0ZDBjOWIwYjE3Njc=" | base64 --decode
2ab96390c7dbe3439de74d0c9b0b1767
```

However for the second row the `“title”` is a base64 decoded version of its associated string but the `“subtitle”` is `h.c()`.

```jsx
contentValues.put("title", Base64.decode("VGhlIGZsYWcgaXMgYWxzbyBhIHBhc3N3b3JkIQ==", 0));
contentValues.put("subtitle", h.c());
writableDatabase.insert("Thisisatest", null, contentValues);
```

Base64 decoding the `“title”` we receive the following:

```jsx
title: echo "VGhlIGZsYWcgaXMgYWxzbyBhIHBhc3N3b3JkIQ==" | base64 --decode
The flag is also a password!
```

Wanting to know what is being supplied from `h.c()` I decided to use use sqlite3 to interact with the database.

```jsx
emu64xa:/data/data/b3nac.injuredandroid/databases # sqlite3 Thisisatest.db
SQLite version 3.32.2 2021-07-12 15:00:17
Enter ".help" for usage hints.
sqlite> .tables
Thisisatest       android_metadata
sqlite> SELECT * FROM Thisisatest;
1|The flag hash!|2ab96390c7dbe3439de74d0c9b0b1767
2|The flag is also a password!|9EEADi^^:?;FC652?5C@:5]7:C632D6:@]4@>^DB=:E6];D@?
```

So the string `2ab96390c7dbe3439de74d0c9b0b1767` from row 1 is a hash. Lets see if we can crack it. Using hashid we can quickly verify that it is likely to be a MD5 hash.

```jsx
hashid 2ab96390c7dbe3439de74d0c9b0b1767
Analyzing '2ab96390c7dbe3439de74d0c9b0b1767'
[+] MD2 
[+] MD5 
[+] MD4
```

Thanks to CrackStation we didn’t even need to open hashcat. 

```jsx
Hash	                           Type	Result
2ab96390c7dbe3439de74d0c9b0b1767	md5	hunter2
```

Now onto row 2 we could look into `h.c()` but lets see if we can get an easy win first. Using ciphey we uncover the plaintext version of the string.

```jsx
docker run -it --rm remnux/ciphey "9EEADi^^:?;FC652?5C@:5]7:C632D6:@]4@>^DB=:E6];D@?"
Possible plaintext: 'https://injuredandroid.firebaseio.com/sqlite.json' (y/N): y
╭───────────────────────────────────────────────────────────────────────────╮
│ The plaintext is a Uniform Resource Locator (URL)                         │
│ Formats used:                                                             │
│    rot47:                                                                 │
│     Key: 47Plaintext: "https://injuredandroid.firebaseio.com/sqlite.json" │
╰───────────────────────────────────────────────────────────────────────────╯
```

Lastly, running opening the URL we can get the last part of the puzzle.

```jsx
 curl https://injuredandroid.firebaseio.com/sqlite.json
"S3V3N_11"
```

# Flag Eight: AWS

`FlagEightLoginActivity`

On top of searching for AWS and the string names I ran trufflehog but did not find any strings that match any AWS patterns or anything similar. For sanity sake I looked at another write-up of this app which in their case they found the AWS credentials within strings.xml. 

Looking into strings.xml we find two spaces for AWS credentials to be in but no info is shown for these lines. For whatever reason those credentials are no longer in this app.

```jsx
<string name="AWS_ID" />
<string name="AWS_SECRET" />
```

# Flag Nine: Firebase

`FlagNineFirebaseActivity`

This activity gives a rather helpful hint, "Use the .json trick with database url”. A quick Google search we can find an article provided in hacktricks. [https://cloud.hacktricks.xyz/pentesting-cloud/gcp-security/gcp-services/gcp-databases-enum/gcp-firebase-enum](https://cloud.hacktricks.xyz/pentesting-cloud/gcp-security/gcp-services/gcp-databases-enum/gcp-firebase-enum)

First up we will need to find the Firebase URL e can go ahead and grep for Firebase or alternatively if we provide the APK to MobSF the Firebase URL will automatically be located for us. Either way the Firebase URL is located in the strings.xml file.

`<string name="firebase_database_url">https://injuredandroid.firebaseio.com</string>`

Now that we have the URL lets see what we need to trigger the flag. `b()` shows us that if `this.f1456b`, which is our user supplied string, is equal to the string from `aVar.c()` then the flag will be triggered. Since `aVar` is from the Firebase database lets look into the database itself.

```jsx
public void b(com.google.firebase.database.a aVar) {
            d.s.d.g.e(aVar, "dataSnapshot");
            if (!d.s.d.g.a(this.f1456b, (String) aVar.c())) {
                Toast.makeText(FlagNineFirebaseActivity.this, "Try again! :D", 0).show();
                return;
            }
            FlagsOverview.J = true;
            j jVar = new j();
            Context applicationContext = FlagNineFirebaseActivity.this.getApplicationContext();
            d.s.d.g.d(applicationContext, "applicationContext");
            jVar.b(applicationContext, "flagNineButtonColor", true);
            FlagNineFirebaseActivity.this.G();
        }
```

Looking into `FlagNineFirebaseActivity` the string `"ZmxhZ3Mv"` is being decoded and used as the reference directory.

```jsx
public FlagNineFirebaseActivity() {
        byte[] decode = Base64.decode("ZmxhZ3Mv", 0);
        this.y = decode;
        d.s.d.g.d(decode, "decodedDirectory");
        Charset charset = StandardCharsets.UTF_8;
        d.s.d.g.d(charset, "StandardCharsets.UTF_8");
        this.z = new String(decode, charset);
        com.google.firebase.database.f b2 = com.google.firebase.database.f.b();
        d.s.d.g.d(b2, "FirebaseDatabase.getInstance()");
        com.google.firebase.database.d d2 = b2.d();
        d.s.d.g.d(d2, "FirebaseDatabase.getInstance().reference");
        this.A = d2;
        com.google.firebase.database.d h = d2.h(this.z);
        d.s.d.g.d(h, "database.child(refDirectory)");
        this.B = h;
    }
```

Base64 decode the string and append to the Firebase URL.

```jsx
echo "ZmxhZ3Mv" | base64 --decode
flags/

curl https://injuredandroid.firebaseio.com/flags/.json
"[nine!_flag]"
```

Looking into `submitFlag()` we can see that our string is being base64 decoded so we will need to bse64 encode our string and then supply the answer.